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
$ vi src/App.js
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

##### src/App.js
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

##### src/App.js
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
  const [weatherInfo, setWeatherInfo] = useState([]);

  // 気象情報APIコールを実行
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

##### src/App.js
```jsx

...

const App = () => {
  const location = locationList[0];
  const [weatherInfo, setWeatherInfo] = useState([]);

  // 気象情報APIコールを実行
  useEffect(() => {
    (async() => {
      try {
        const res = await weatherClient.get(
          `${openMeteoApiBase}?timezone=Asia/Tokyo&latitude=${location.lat}&longitude=${location.lon}&hourly=weather_code`
        );
        /* 下記削除 */
        const weatherData = res.data;
        setWeatherInfo(weatherData);
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
  0: '☀️快晴',
  1: '☀️晴れ',
  2: '☁️薄曇り',
  3: '☁️曇り',
  45: '🌫️霧',
  48: '🌫️氷霧',
  51: '☔薄い霧雨',
  53: '☔霧雨',
  55: '☔濃い霧雨',
  56: '☔薄い着氷性の霧雨',
  57: '☔濃い着氷性の霧雨',
  61: '☔小雨',
  63: '☔雨',
  65: '☔大雨',
  66: '☔弱い氷雨',
  67: '☔強い氷雨',
  71: '❄️小雪',
  73: '❄️雪',
  75: '❄️大雪',
  77: '❄️霧雪',
  80: '☔にわか雨',
  81: '☔通り雨',
  82: '☔集中豪雨',
  85: '❄️弱いにわか雪',
  86: '❄️強いにわか雪',
  95: '⚡雷雨',
  96: '⚡霰を伴う雷雨',
  99: '⚡雹を伴う雷雨',
}

// このファイルの内容をほかのファイルから呼び出すための設定
module.exports = {
  weatherNames
}
```

参考:
https://note.com/note50/n/n39c1dc054a08

### 気象名の表示
それでは画面を作成していきます。
まずはタイトルと、気象情報一覧のテーブルから作成しましょう。
##### src/App.js
```jsx

...

const App = () => {

...

  return (
    {/* 下記削除 */}
    <div style={{ whiteSpace: 'pre-wrap' }}>{JSON.stringify(weatherInfo, null, 2)}</div>
    {/* 下記追加 */}
    <div>
      <h1>{location.jpName} の天気</h1>
      <div>
        <table id='weather-table' border={1}>
          <tbody>
            {/* 取得した1時間おきの気象データを順番に表示します */}
            {weatherInfo.map((info) => (
              <tr key={info.datetime}>
                <td>{info.datetime}</td>
                <td>{info.weatherCode}</td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
    {/* ここまで */}
  );
}
```

少しWebページらしくなりましたね
(img)ui_title_table

ここで先ほど設定した気象名を使用し、気象コードを気象名に変換して表示します。
##### src/App.js
```jsx
import { useEffect, useState } from 'react';
import axios from 'axios';
/* 下記追加 */
import { weatherNames } from './weather_names';
/* ここまで */

...

const App = () => {

...

  return (
    <div>
      <h1>{location.jpName} の天気</h1>
      <div>
        <table border={1}>
          <tbody>
            {weatherInfo.map((info) => (
              <tr key={info.datetime}>
               <td>{info.datetime}</td>
                {/* 下記削除 */}
                <td>{info.weatherCode}</td>
                {/* 下記追加 */}
                <td>{weatherNames[info.weatherCode]}</td>
                {/* ここまで */}
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
}
```

ついでに時刻情報も読みにくいので、表記を切り替えます。
date-fnsは、日時データの操作に長けた便利なライブラリです。
今回はformat関数をdate-fnsから読み込んで使いましょう。
##### src/App.js
```jsx
import { useEffect, useState } from 'react';
import axios from 'axios';
/* 下記追加 */
import { format } from 'date-fns';
/* ここまで */
import { weatherNames } from './weather_names';

...

const App = () => {

...

  return (
    <div>
      <h1>{location.jpName} の天気</h1>
      <div>
        <table id='weather-table' border={1}>
          <tbody>
            {weatherInfo.map((info) => (
              <tr key={info.datetime}>
                {/* 下記削除 */}
                <td>{info.datetime}</td>
                {/* 下記追加 */}
                <td>{format(new Date(info.datetime), 'MM/dd - HH:mm')}</td>
                {/* ここまで */}
                <td>{weatherNames[info.weatherCode]}</td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
}
```

最低限、天気予報として必要そうな表示になりましたね！
(i㎎)ui_table_jp

## 表示できる地点を増やそう
現在は東京の天気のみ表示されておりますが、表示できる地点を増やしてみましょう。

### 地点選択機能
最初のほうに定義していた、locationListに別の地点情報を追加します。

加えて地点の切り替えは画面上から行いたいので、地点を選択できるセレクトボックスを設置します。
##### src/App.js
```jsx

...

const locationList = [
  { enName: 'tokyo', jpName: '東京', lat: 35.689, lon: 139.692 }, // 末尾にカンマを足す
  /* 下記追加 */
  { enName: 'osaka', jpName: '大阪', lat: 34.686, lon: 135.520 },
  { enName: 'saga', jpName: '佐賀', lat: 33.249, lon: 130.300 }
  /* ここまで */
];

const App = () => {

...

  return (
    <div>
      {/* 下記追加 */}
      <div>
        <select id='location-select'>
          {/* locationListの要素数分の選択肢を出します */}
          {locationList.map((lo) => (
            <option key={lo.enName}>{lo.jpName}</option>
          ))}
        </select>
      </div>
      {/* ここまで */}
      <h1>{location.jpName} の天気</h1>

...

```
地点選択用のセレクトボックスが表示されたかと思います。
(img)ui_location-select

### 選択された地点ごとの気象データ取得
地点が選択されるようになりましたが、現状では表示されている気象情報が変化しません。
選択した地点に応じて、気象情報の取得処理(useEffectの箇所)を再実行する必要があるためですね。

##### src/App.js
```jsx

...

const App = () => {
  /* 下記削除 */
  const location = locationList[0];
  /* 下記追加 */
  const [location, setLocation] = useState(locationList[0]);
  /* ここまで */
  const [weatherInfo, setWeatherInfo] = useState([]);

  /* 下記追加 */
  // 地点選択時のアクション
  const onChangeLocation = (event) => {
    const currentLocationData = locationList.find((lo) => event.target.value === lo.enName);
    setLocation(currentLocationData);
  }
  /* ここまで */

  // 気象情報APIコールを実行
  useEffect(() => {

...

  }, [location]) // locationを第2引数に追加(location更新時にuseEffectを再実行)

  return (
    <div>
      <div>
        {/* selectが変更された際にonChangeLocation関数を作動させます */}
        <select id='location-select' onChange={onChangeLocation}>
          {locationList.map((lo) => (
            <option key={lo.enName}>{lo.jpName}</option>
          ))}
        </select>
      </div>
      <h1>{location.jpName} の天気</h1>

...

```
これで様々な地点の天気情報が表示できましたね。
アプリケーションの最低限の機能が完成となります。

## デザイン設定
最後に、作成したアプリケーションのデザインを調整しましょう。
今回の講義では、デザイン面であるCSSの設定等は詳しく触れません。
興味があればご自身で調べてみてください。

まずはreact側にcssの設定ポイントとなるクラス名を指定します
##### src/App.js
```jsx

...

  return (
    <div>
      <div className='center-item'> {/* className追加 */}
        <select id='location-select' onChange={onChangeLocation}>
          {locationList.map((lo) => (
            <option key={lo.enName} value={lo.enName}>{lo.jpName}</option>
          ))}
        </select>
      </div>
      <h1 className='center-item'>{location.jpName} の天気</h1> {/* className追加 */}
      <div className='center-item'> {/* className追加 */}
        <table id='weather-table' border={1} >
          <tbody>
            {weatherInfo.map((info) => (
              <tr key={info.datetime}>
                <td>{format(new Date(info.datetime), 'MM/dd - HH:mm')}</td>
                <td>{weatherNames[info.weatherCode]}</td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
}
```

続いてcssファイルにcssを記述します。
コピペでOK

```sh
$ vi src/index.css
```
##### src/index.css
```css
/*
元の設定はすべて削除
*/

body {
  margin: 0;
  padding: 2em;
  font-family: 'Segoe UI';
  background-color: lightcyan;
}

td {
  padding: 0.5em;
}

.center-item {
  display: flex;
  justify-content: center;
}

#location-select {
  padding: 0.5em 1em;
  width: 200px;
  background-color: lemonchiffon;
  border: 2px solid orange;
  border-radius: 10px;
}

#weather-table {
  background-color: white;
}
```
これで今回のアプリケーションは完成です。
お疲れさまでした。
