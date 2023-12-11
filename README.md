# document-test
## システム設定値を修正
```sh
$ echo fs.inotify.max_user_watches=16384 | sudo tee -a /etc/sysctl.conf
fs.inotify.max_user_watches=24288
$ sudo sysctl -p
fs.inotify.max_user_watches = 24288
$ cat /proc/sys/fs/inotify/max_user_watches
24288
$
```

## Reactアプリケーション作成
```sh
$ npx create-react-app weather-app
# create-react-appコマンドがインストールされていないと下記が出ます。
# 「y」を入力しEnterでOK
Need to install the following packages:
  create-react-app@5.0.1
Ok to proceed? (y) 

# 下記のような表示が出て、初期テンプレートのインストールが始まります。
Creating a new React app in /home/ec2-user/weather-app.

# 下記のように出ればReactアプリケーションのテンプレートが作成完了です。
Success! Created weather-app at /home/ec2-user/weather-app
Inside that directory, you can run several commands:

  npm start
    Starts the development server.

  npm run build
    Bundles the app into static files for production.

  npm test
    Starts the test runner.

  npm run eject
    Removes this tool and copies build dependencies, configuration files
    and scripts into the app directory. If you do this, you can’t go back!

We suggest that you begin by typing:

  cd weather-app
  npm start

Happy hacking!
```

## ライブラリインストール
```sh
$ npm install @mui/material @emotion/react @emotion/styled axios date-fns
# 下記のような表示が出ればインストールOK(数値に差異があるのはOK)

added 43 packages, and audited 1575 packages in 14s
```

## Hello World表示
アプリケーション作成後のメッセージで表示されているコマンドを使用し、試しにReactアプリケーションを立ち上げる
```sh
$ cd weather-app
$ npm start

# 下記のように出ればアプリケーションの立ち上げ成功です。
# 表示されているhttp://localhost:3000にアクセスしてみましょう。
Compiled successfully!

You can now view weather-app in the browser.

  Local:            http://localhost:3000
  On Your Network:  http://192.168.94.126:3000

Note that the development build is not optimized.
To create a production build, use npm run build.

webpack compiled successfully
```
下記のようにReactロゴの画面が出れば成功です。
(img)react_skelton

## アプリケーションを編集
```sh
$ vi src/app.js
```
##### src/app.js
```jsx
/* 下記削除 */
import logo from './logo.svg';
import './App.css';
/* ここまで */

// ↓の1行は書き方の違いだけなので、自由です
-function App() {
+const App = () => {
  return (
    /* 下記追加 */
    <div>Hello, React JS.</div>
    /* ここまで */
    
    /* 下記削除 */
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <p>
         Edit <code>src/App.js</code> and save to reload.
        </p>
        <a
          className="App-link"
          href="https://reactjs.org"
          target="_blank"
          rel="noopener noreferrer"
        >
          Learn React
        </a>
      </header>
    </div>
    /* ここまで */
  );
}

export default App;
```

いわゆるハローワールドです。
(img)hello_react

## 東京の天気を取得しよう
天気予報アプリケーションの作成を始めます。  
ひとまずは、東京都の天気を試しに表示してみます。

気象情報の取得には下記のAPIを使用します。
OpenMeteo API
https://open-meteo.com/

### 地点情報定義
このAPIの仕様として、気象情報の取得に緯度/経度が必要となります。
そのため下記のように東京都の情報を定義します。

緯度/経度情報:
http://agora.ex.nii.ac.jp/digital-typhoon/search_place.html.ja

##### src/app.js
```jsx
/* 下記追加 */
const locationList = [
  { enName: 'tokyo', jpName: '東京', lat: 35.689, lon: 139.692 }
];
/* ここまで */

const App = () => {
  /* 下記追加 */
  const location = locationList[0];
  /* ここまで */

  return (
    <div>Hello, React JS.</div>
  );
}
```

東京都の地点情報を配列の要素として定義したのは、後ほど別地点の情報も追加できるようにしたいためです。
事前に「複数地点の気象情報を取得する」と要件定義しているので、このように先回りした実装を行います。

### 気象情報取得APIコール
実際に東京都の気象情報をAPIで取得してみます。
アプリケーション内からコールするURLは下記です。
`https://api.open-meteo.com/v1/forecast?latitude=35&longitude=135&hourly=weather_code&timezone=Asia%2FTokyo`

##### src/app.js
```jsx
/* 下記追加 */
import { useEffect, useState } from 'react';
import axios from 'axios';

const weatherClient = axios.create({
  headers: {
    'Content-Type': 'application/json'
  }
});

const openMeteoApiBase = 'https://api.open-meteo.com/v1/forecast'; // 気象情報API
/* ここまで */

const locationList = [
  { enName: 'tokyo', jpName: '東京', lat: 35.689, lon: 139.692 }
];

const App = () => {
  const location = locationList[0];
  /* 下記追加 */
  const [weatherInfo, setWeatherInfo] = useState({});

  useEffect(() => {
    (async() => {
      try {
        // APIコールし気象情報取得
        const res = await weatherClient.get(
          `${openMeteoApiBase}?timezone=Asia/Tokyo&latitude=${location.lat}&longitude=${location.lon}&hourly=weather_code`
        );
        const weatherData = res.data;
        // stateに気象データをセット
        setWeatherInfo(weatherData);
      } catch (error) {
        alert(error.message);
      }
    })();
  }, [])
  /* ここまで */

  return (
    {/* 下記削除 */}
    <div>Hello, React JS.</div>
    {/* ここまで */}

    {/* 下記追加 */}
    <div style={{ whiteSpace: 'pre-wrap' }}>{JSON.stringify(weatherInfo, null, 2)}</div> {/* 気象情報表示 */}
    {/* ここまで */}
  );
}
```
画面にAPIの取得結果が表示されます。
(img)meteo_all

### 取得データ成型
APIから取得したデータには余分な内容があります。
今回必要なデータは"hourly"に入っているデータのみなので、下記のようにして必要なデータのみ抜き出します。

##### src/app.js
```jsx

...

const App = () => {
  const location = locationList[0];
  const [weatherInfo, setWeatherInfo] = useState({});

  useEffect(() => {
    (async() => {
      try {
        const res = await weatherClient.get(
          `${openMeteoApiBase}?timezone=Asia/Tokyo&latitude=${location.lat}&longitude=${location.lon}&hourly=weather_code`
        );
        /* 下記削除 */
        const weatherData = res.data;
        setWeatherInfo(weatherData);
        /* ここまで */
        /* 下記追加 */
        const weatherData = res.data.hourly;
        const weatherDataByTime = weatherData.time.map((time, index) => {
          return {
            datetime: time,
            weatherCode: weatherData.weather_code[index]
          }
        }, {});
        setWeatherInfo(weatherDataByTime);
        /* ここまで */
      } catch (error) {
        alert(error.message);
      }
    })();
  }, [])

  return (
    <div style={{ whiteSpace: 'pre-wrap' }}>{JSON.stringify(weatherInfo, null, 2)}</div>
  );
}
```

(img)meteo_array

## UI設定
気象データが取れました。
しかし今画面に出ているのは生のAPIデータなので、非常に見にくいです。
そのため続いては画面UIの設定をしていきましょう。

### 気象コードを気象名と対応させる
画面設定の前に、設定が必要な要素があります。
現在は気象の名前が取得できておらず、気象コードしかない状態です。
気象名についてはAPIから取得することはできないので、自分で設定を行いましょう。
```sh
$ vi src/weather_name.js
```
### weather_name.js
```javascript
// 下記コピペでOKです。
const weatherNames = {
  0: '快晴',
  1: '晴れ',
  2: '薄曇り',
  3: '曇り',
  45: '霧',
  48: '氷霧',
  51: '薄い霧雨',
  53: '霧雨',
  55: '濃い霧雨',
  56: '薄い着氷性の霧雨',
  57: '濃い着氷性の霧雨',
  61: '小雨',
  63: '雨',
  65: '大雨',
  66: '弱い氷雨',
  67: '強い氷雨',
  71: '小雪',
  73: '雪',
  75: '大雪',
  77: '霧雪',
  80: 'にわか雨',
  81: '通り雨',
  82: '集中豪雨',
  85: '弱いにわか雪',
  86: '強いにわか雪',
  95: '雷雨',
  96: '霰を伴う雷雨',
  99: '雹を伴う雷雨',
}

// このファイルの内容をほかのファイルから呼び出すための設定
module.exports = {
  weatherNames
}
```

参考:
https://note.com/note50/n/n39c1dc054a08


