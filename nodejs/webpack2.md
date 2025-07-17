# Webpack

## 1. Webpack이란?

Webpack은 \*\*모듈 번들러(module bundler)\*\*입니다. 애플리케이션을 구성하는 다양한 리소스(JS, CSS, 이미지 등)를 모두 모듈처럼 취급하여 **의존성 그래프를 생성하고, 하나 이상의 번들 파일로 패키징**합니다.

> 📌 정리: Webpack은 "프로젝트 전반의 자산을 하나의 파일로 구성해주는 자동화 도구"입니다.

---

## 2. 왜 Webpack이 필요한가?

### ✅ 문제 상황 (전통적 웹 개발)

* 여러 JS/CSS 파일을 `<script>`, `<link>`로 각각 로딩
* 파일이 많아지면 네트워크 요청 증가 → 성능 저하
* 모듈화가 어려움 → 글로벌 스코프 오염, 유지보수 어려움

### ✅ Webpack 도입 후

* `import` 문을 사용한 모듈 기반 개발 가능
* 필요 시 자동 코드 분할
* 모든 리소스를 하나의 번들로 묶어 최적화
* 개발과 배포 환경 구분

---

## 3. 핵심 개념

| 개념     | 설명                                            |
| ------ | --------------------------------------------- |
| Entry  | 앱 실행의 진입점(보통 `src/index.js`)                  |
| Output | 번들 파일의 이름과 경로 설정                              |
| Loader | JS 외 파일(CSS, 이미지 등)을 처리하는 규칙                  |
| Plugin | 번들 이후 작업(HTML 생성, 압축 등)을 담당하는 확장 도구           |
| Mode   | `development` / `production` 모드에 따라 최적화 전략 다름 |

---

## 4. 최소 설정 예시 (webpack.config.js)

```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  mode: 'development',
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
    clean: true,
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: 'babel-loader',
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      },
    ],
  },
  plugins: [
    new HtmlWebpackPlugin({ template: './public/index.html' }),
  ],
};
```

---

## 5. 고급 기능

### 📦 코드 분할 (Code Splitting)

* `import()` 구문으로 비동기 로딩

```js
import('./Page').then(Page => {/* ... */});
```

### 🌳 트리 쉐이킹 (Tree Shaking)

* 사용하지 않는 모듈 제거
* `sideEffects: false`, `mode: 'production'` 설정 필요

### 🗺️ Source Map

```js
devtool: 'source-map';
```

* 디버깅 시 원본 코드와 번들 간의 매핑 제공

### 🔧 환경변수 처리

```js
new webpack.DefinePlugin({
  'process.env.NODE_ENV': JSON.stringify('production')
});
```

---

## 6. Webpack 실행 명령어

```bash
npx webpack              # 기본 번들 실행
npx webpack --mode production    # 프로덕션 번들링
npx webpack serve        # dev-server 실행 (webpack-dev-server 필요)
```

---

## 7. Webpack과 다른 번들러 비교

| 항목       | Webpack | Vite  | Parcel | Rollup |
| -------- | ------- | ----- | ------ | ------ |
| 설정 유연성   | 매우 높음   | 낮음    | 중간     | 높음     |
| 초기 빌드 속도 | 느림      | 매우 빠름 | 빠름     | 빠름     |
| 코드 분할    | 지원      | 지원    | 지원     | 지원     |
| 생태계 크기   | 매우 큼    | 증가 중  | 작음     | 중간     |
| SSR 지원   | 가능      | 기본 지원 | 제한적    | 제한적    |

---

## 8. 실무 Best Practice

* 🔹 개발 모드: `mode: 'development'`, `devtool: 'eval-source-map'`
* 🔹 배포 모드: `mode: 'production'`, `optimization`, `minify`
* 🔹 코드 분할: React.lazy + dynamic import
* 🔹 번들 분석: `webpack-bundle-analyzer` 사용
* 🔹 캐싱 전략: `[name].[contenthash].js` 활용

---

## ✅ 마무리 요약

* Webpack은 가장 강력한 번들러이며, 자유도가 높음
* 하지만 설정이 복잡하므로 필요에 따라 CRA/Vite로 시작하고, Webpack은 고급 설정이 필요할 때 커스터마이징 도구로 도입하는 것이 좋음
* 대규모 앱에서는 코드 분할, 캐싱, 성능 최적화 등 Webpack의 정교한 기능이 강력한 무기가 됨
