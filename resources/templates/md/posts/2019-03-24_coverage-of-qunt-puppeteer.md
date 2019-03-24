{:title "QUnitでCIしたい その2"
 :layout :post
 :tags  ["node", "qunit", "puppeteer", "nyc"]}

[前回](./2019-03-21_want-to-run-qunit-in-cli.md)、QUnitのユニットテストをpuppeteerでCLI実行できるようにした

今回はcode coverageを計測できるようにする。

### 前回のおさらい

puppeteerでJavaScriptのカバレッジを計測することができる([What's New In DevTools (Chrome 59)  |  Web  |  Google Developers](https://developers.google.com/web/updates/2017/04/devtools-release-notes))のだが、現状利用している[node-qunit-puppeteer](https://www.npmjs.com/package/node-qunit-puppeteer)だと、puppeteerの部分をモジュール外部から触れないので利用できなくて困った、ということろまで書いた。

理想ではユニットテストの実施と同時にカバレッジを計測したいところなのだが、一旦は簡単のために別々に、つまりユニットテストの実行後さらにカバレッジ取得のためにユニットテスを再実行する、というかたちで楽しようと考えた。ユニットテストの実行が軽いうちは2度実行したところで大した負荷にならないというのもあり。

### やったこと

これ↓

[istanbuljs/puppeteer-to-istanbul: given coverage information output by puppeteer's API output a format consumable by Istanbul reports](https://github.com/istanbuljs/puppeteer-to-istanbul)

puppeteerで計測したカバレッジをistanbul(nyc)で利用できる形に書き出すモジュールを使う。

[すでにあるnode-qunit-puppeteerのテストランナー](https://github.com/krymtkts/qunit-trial/blob/master/test/run.js)と一緒には使えないので、カバレッジ計測のためのスクリプトも新たに書く(無駄)。

```js
const pti = require('puppeteer-to-istanbul');
const puppeteer = require('puppeteer');
const path = require('path');

(async () => {
  try {
    const browser = await puppeteer.launch();
    const page = await browser.newPage();

    await page.coverage.startJSCoverage();
    await page.goto(`file://${path.join(__dirname, '/index.html')}`);
    const jsCoverage = await page.coverage.stopJSCoverage();

    pti.write(jsCoverage);
    await browser.close();
  } catch (error) {
    console.error(error);
  }
})();
```

これもほぼサンプルママで使えた。読み込ませるページのパスだけ工夫が必要ってだけ。

あとはカバレッジ計測に成功したらnycのレポート作成を実行するだけでおｋ。これをnpmのタスクにする。

```sh
node test/coverage.js && nyc report --reporter=html
```

これでページを開いたときに読み込まれるJSのカバレッジを計測できた。ただしこのままだと、QUnitやユニットテスト自体のカバレッジも含まれてしまうので、仕事のCIで使う場合なんかには特定のファイルのカバレッジだけを見るか、あるいは除外設定があればいいのだけど。

### まとめ

また今回試した内容は以下のrepoに反映してある

[krymtkts/qunit-trial: sandbox to enhance legacy QUnit test.](https://github.com/krymtkts/qunit-trial)

2回ユニットテストを実行していて無駄感があるので、node-qunit-puppeteerに手を入れることも検討していいかも🤔

続く