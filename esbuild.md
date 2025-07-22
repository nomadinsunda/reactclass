# ⚡ esbuild: 초고속 번들러 & 트랜스파일러

## ✅ esbuild란?

**esbuild**는

> Go 언어로 작성된 **초고속 트랜스파일러 & 번들러**입니다.

JS/TS 코드의 변환, 압축, 번들링, 최적화를 모두 지원하며,
Webpack이나 Babel보다 **최소 10\~100배 빠른 속도**를 자랑합니다.

---

## 📦 개발자 관점 요약

| 항목    | 내용                                      |
| ----- | --------------------------------------- |
| 작성 언어 | **Go** (멀티스레드 + 네이티브 바이너리)              |
| 사용 목적 | 트랜스파일, 번들, 압축, 모듈 시스템 변환                |
| 지원 언어 | JS, JSX, TS, TSX, JSON                  |
| 사용처   | Vite, Bun, Snowpack, Turbopack, Metro 등 |
| 특징    | 빠른 속도, ESM 최적화, 병렬 처리 가능                |

---

# 🎯 esbuild의 주요 기능

| 기능             | 설명                                   |
| -------------- | ------------------------------------ |
| **트랜스파일링**     | TypeScript → JavaScript, JSX → JS 변환 |
| **번들링**        | 여러 파일을 하나로 번들링 (CommonJS ↔ ESM 포함)   |
| **압축(minify)** | Dead code 제거, 코드 난독화, 축소             |
| **코드 스플리팅**    | `dynamic import()` → 청크 분리 가능        |
| **서버 빌드 지원**   | ESM → Node.js용 CJS 변환 지원             |
| **플러그인 API**   | 커스텀 플러그인 개발 가능 (Vite에서도 활용됨)         |

---

# 🚀 esbuild vs 기존 도구

| 비교 항목        | Babel        | Webpack | Rollup | **esbuild**      |
| ------------ | ------------ | ------- | ------ | ---------------- |
| 목적           | 트랜스파일러       | 번들러     | 번들러    | 둘 다 (all-in-one) |
| 작성 언어        | JS           | JS      | JS     | **Go**           |
| 속도           | 느림           | 느림      | 중간     | **매우 빠름**        |
| Tree-shaking | ❌ Babel 자체 X | O (약함)  | ✅ 강력   | ✅ 매우 빠름          |
| 사용 편의        | 복잡           | 복잡      | 중간     | 간단               |

> ✅ **esbuild은 Babel + Webpack을 대체**할 수 있을 정도로 강력하지만, **세밀한 제어는 Rollup보다 약합니다.**

---

# 🧪 실전 예제: esbuild 단독 사용

```bash
# 설치
npm install -D esbuild
```

```bash
# 명령어로 TypeScript 트랜스파일
npx esbuild src/app.ts --outfile=dist/app.js --bundle --minify --sourcemap
```

### 예제 설명

| 옵션                | 설명                  |
| ----------------- | ------------------- |
| `--bundle`        | 모든 의존성을 하나의 번들로     |
| `--minify`        | 축소/난독화              |
| `--sourcemap`     | 디버깅용 소스맵 생성         |
| `--target=es2017` | 출력되는 JS 표준 버전 설정 가능 |

---

# 🔌 esbuild in **Vite**

Vite는 다음과 같은 방식으로 esbuild를 활용합니다:

| 사용 단계                | esbuild 역할                                 |
| -------------------- | ------------------------------------------ |
| 개발 서버 (`vite dev`)   | `.ts`, `.tsx`, `.jsx` 파일을 실시간 트랜스파일        |
| 플러그인 처리              | JSX 변환 등 일부 플러그인도 esbuild 기반               |
| 최종 빌드 (`vite build`) | Rollup이 기본 번들링하지만 내부 일부 트랜스파일은 여전히 esbuild |
| JSX fast refresh     | React + Vite에서는 JSX 핫리로드에도 esbuild 사용      |

Vite의 속도가 Webpack보다 빠른 이유 중 하나가 바로 **esbuild 덕분**입니다.

---

# ⚙️ 내부 구조 (아키텍처)

```plaintext
Input: TypeScript / JSX / TSX / ESM
        ↓
  [esbuild Loader]
        ↓
 AST 변환 (Go 기반 분석)
        ↓
 트리 쉐이킹 + 최적화 + 번들링
        ↓
Output: Optimized JavaScript (ESM or CJS)
```

* AST를 Go에서 직접 파싱 → JS보다 빠름
* 멀티스레드 → CPU 코어 최대 활용

---

# 🧩 플러그인 시스템 (esbuild 자체 플러그인 API)

```js
// 간단한 esbuild 플러그인 예시
const myPlugin = {
  name: 'replace-env',
  setup(build) {
    build.onLoad({ filter: /\.js$/ }, async (args) => {
      let contents = await fs.promises.readFile(args.path, 'utf8');
      contents = contents.replace('process.env.NODE_ENV', '"production"');
      return { contents, loader: 'js' };
    });
  },
};
```

> esbuild 자체로도 Rollup 수준의 커스터마이징 가능

---

# ❌ esbuild의 한계

| 항목                       | 한계점                   |
| ------------------------ | --------------------- |
| 코드 분해 제어                 | Rollup보다 미세 제어 어려움    |
| 플러그인 생태계                 | Rollup보다 적음           |
| CSS/HTML 핸들링             | 기본 지원 없음 (추가 플러그인 필요) |
| SSR, dynamic import 메타정보 | 제한적                   |

> 그래서 \*\*Vite는 개발 시 `esbuild`, 빌드시 `Rollup`\*\*을 사용하여
> 양쪽의 장점을 모두 누립니다.

---

# ✅ 결론 요약

| 항목         | 설명                                 |
| ---------- | ---------------------------------- |
| 핵심 개념      | Go 기반 초고속 트랜스파일러 + 번들러             |
| Vite에서의 역할 | 개발 환경 속도 극대화 (`vite dev`)          |
| 비교         | Babel/Webpack보다 수십 배 빠름            |
| 단점         | 제어력은 Rollup보다 부족                   |
| 실무 팁       | React/TS 기반 개발 시 HMR 속도 비약적 개선에 기여 |

