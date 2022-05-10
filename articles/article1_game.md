---
title: "情報系の大学に入ったので春休み中にReact+TSでマインスイーパーとテトリス作ってみた"
emoji: "💻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [react, typescript]
published: true
---
はじめまして。大学2年の**Yoshi**と申します。
今回が初めての記事で、マインスイーパーとテトリスの解説だけでなく自己紹介やゲームを作った経緯なども合わせて載せています。
これから優れたエンジニアになるために学んでいく様々なことをZennに記録として残していこうと思いますので、どうぞよろしくお願いします！

# 自己紹介
東洋大学情報連携学部2年のYoshiと言います。
プログラミングは大学に入ってから真剣に学び始めました。
子供の時から何か面白いものを作りたいという欲があり、プログラミングを学ぶことを決意しました。

# 大学の講義で学んだこと
講義では主に**Python**や**JavaScript**、**アルゴリズム**の基礎的なことを学びました。また、**Django**を使ってWebアプリケーションを作ったりしました。
課題やテストは問題なくこなしています。

# React+TSとの出会い
大学で**TypeScript**を学ぶ非公式サークルができ、大学の講義の内容だけでは成長をあまり感じることができず、もっといろいろなプログラミングを学びたいと思い参加し、学び始めました。
:::message
同じサークルの仲間もこの記事と同じような内容で記事をZennに上げています
https://zenn.dev/kado17/articles/article1_introduction
:::

# なぜゲームを作ったのか
もともと学ぼうと思ったのは、web系のプログラミングでした。しかし、そんな中でゲーム作りを選んだ理由は**フレームワークの使い方**と**アルゴリズム**を同時に習得するのに良く、デザインがほとんど不要で**コーティングに注力**できると思ったからです。そしてその中でも特に、ユーザーの操作で状態が著しく変化するプログラムを書く練習にもなると考えました。

# マインスイーパーの解説
まず最初に**マインスイーパー**というゲームを製作しました。
ぜひ遊んでみてください！
:::message
マインスイーパーは地雷原から地雷の場所を特定し、地雷以外の全てのマスを開けていくゲームです。クリックした時に出てくる数字は周り１マスにある地雷の数です。(地雷は合計10個あります)
操作方法：基本左クリック、右クリックで旗を立てることができ、地雷であると考えられるものに目印をつけられます
:::
https://yoshi-program.github.io/minesweeper/
https://github.com/Yoshi-program/minesweeper/blob/main/pages/index.tsx
↓こちらを参考にしました。
https://minesweeper.online/
特に難しかったのは、クリックした際、周りに一つも爆弾がない時に白マス(周りに地雷がないマスのことを私は白マスと呼んでいます)が連鎖するプログラムです。
![白マス連鎖](https://media.giphy.com/media/vrmtczKlznL0U4LuuR/giphy.gif =300x)
いろいろな方法で試みましたが上手くいかず苦労しましたが、最終的には**連鎖できるマスをリストに追加し続けFor文でゲームボードを書き換える**ことで上手くいきました。
もっと具体的には`CountBombs`で周りにある地雷を数える関数や`ListofAround`で周りの座標をリスト化する関数を使うことで白マスを連鎖させることができました。
```js:index.tsx
// クリックした座標に地雷がないとき
if (!existBomb) {
      let NumBombs = 0
      // 周りの地雷を数える
      NumBombs = CountBombs(x, y, NewBombs) 
      newBoard[y][x] = NumBombs
      setRestBlock((c) => c + 1)

      // 白マス連鎖(周りの地雷が0個だったら)
      if (NumBombs === 0) {
        let NewNumBombs = 0
        // クリックした周りの座礁をリスト化する
        const Coordinate = ListofAround(x, y)
        for (const c of Coordinate) {
          NewNumBombs = CountBombs(c.x, c.y, NewBombs)
          // まだ開いていないマスだったらNewNumBombsの数字で開く
          if (newBoard[c.y][c.x] === 9) {
            newBoard[c.y][c.x] = NewNumBombs
            setRestBlock((c) => c + 1)
          }
          // 地雷が0個だったらその周りの座標をリストに追加する
          if (NewNumBombs === 0) {
            for (const nc of ListofAround(c.x, c.y))
              if (!Coordinate.some((nb) => nb.x === nc.x && nb.y === nc.y)) {
                Coordinate.push({ x: nc.x, y: nc.y })
              }
          }
        }
      }
    }
```

また、React hooksである`useState`や`useEffect`の使い方を学び、JavaScriptの書き方も上手くなりました。
`useState`ではゲームクリアやゲームオーバーの状況管理やゲームボードの状態管理などを行いました。
```js:index.tsx
//ゲームクリアの状態管理
const [gameClear, setGameClear] = useState(false)
```
`useEffect`は時間の計測に使いました。`setInterval`を使うことで毎秒`count`が増えるようになっています。また、ゲームの状況に応じてカウントするかを判定するようにしています。
```js:index.tsx
// 時間の計測
  const [count, setCount] = useState(0)
  useEffect(() => {
    // ゲームの状況に応じてカウントするかを判定する
    if (gameStart && !gameOver && !gameClear) {
      const interval = setInterval(() => {
        setCount((c) => c + 1)
      }, 1000)
      return () => clearInterval(interval)
    }
  }, [gameStart, gameOver, gameClear])
```

# テトリスの解説
次に作ったのが**テトリス**です。
![テトリス](/images/article1/tetris.png =500x)
ぜひ遊んでみてください！
:::message
5ライン消すごとに落ちる速度、固定にかかる時間が速くなります
操作方法：矢印 ← ↓ → が移動、矢印 ↑ 、X が右回転、Z, 左Ctrlで左回転、スペースで一気に下まで、シフトでHoldができます。また、画面のStop!を押すことで一時停止できます。
:::
https://yoshi-program.github.io/tetris/
https://github.com/Yoshi-program/tetris/blob/main/pages/index.tsx
この製作では`useMemo`や`useCallback`などの新しいことを学んだり、`uesEffect`での時間に応じたプログラムを書くことなどの練習になりました。

難しかったのは**テトリミノの扱い**です。画面外にいかないようにしたり、重ならないようにするプログラムを思いつくのが大変でした。
引数にXY座標、テトリミノの状態を与えることで、その位置が画面内かどうかと重なっているかを判定して、重なっていたら`true`,そうでなければ`false`を返す仕組みになっています。
```js:index.tsx
// 動かせるかを判定する関数
const checkCordinate = (cx: number, cy: number, tetromino: number[][]): boolean => {
    for (let y = 0; y < tetromino.length; y++) {
      for (let x = 0; x < tetromino[y].length; x++) {
        if (tetromino[y][x] > 0 && board[y + cy][x + cx] > 0) {
          return true
        }
      }
    }
    return false
  }
```

また、回転時の処理が、周りに障害物があると上手くいかず、最後まで課題として残りましたが、無事にプログラムを書けました。↓
テトリミノの回転状況によって処理が変わるため、三項演算子でプログラムが複雑っぽくなっています。しかし、回転した時に周りにそのテトリミノが存在できる空間があるかを、For文で`moveX`, `moveY`を一定まで増やして判定し、空間があった場合にその`moveX`, `moveY`分動いてから、その回転の処理を行うというシンプルなプログラムです。

```js:index.tsx
//回転させる関数
  const changeRotate = (checkRight:boolean): void => {
    for (let moveY = 0; moveY < 3; moveY++) {
      for (let moveX = 0; moveX < tetromino[0].length - 1; moveX++) {
        if (
          !checkCordinate(x + moveX, y - moveY, tetromino[checkRight ? (rotateNumber < 3 ? rotateNumber + 1 : 0) : (rotateNumber > 0 ? rotateNumber - 1 : 3)])
        ) {
          X((c) => c + moveX)
          Y((c) => c - moveY)
          setRotateNumber(checkRight ? ((c) => (c < 3 ? c + 1 : (c = 0))) : ((c) => (c > 0 ? c - 1 : (c = 3))))
          return
        }
      }
      for (let moveX = -1; moveX > -(tetromino[0].length - 1); moveX--) {
        if (
          !checkCordinate(x + moveX, y - moveY, tetromino[checkRight ? (rotateNumber < 3 ? rotateNumber + 1 : 0) : (rotateNumber > 0 ? rotateNumber - 1 : 3)])
        ) {
          X((c) => c + moveX)
          Y((c) => c - moveY)
          setRotateNumber(checkRight ? (c) => (c < 3 ? c + 1 : (c = 0)) : ((c) => (c > 0 ? c - 1 : (c = 3))))
          return
        }
      }
    }
  }
```
このほかには`useMemo`を使ってゲームボードの状態を管理しました。
`changeBoard`でゲームボードの状態を持ってきて、`slice`で画面に映る上下の範囲を指定し、`filter`で両端の１列をなくすという処理をしています。(両端に9を置くことでテトリミノが画面外に行かないようにプログラムしています)
```js:index.tsx
// 画面に映すゲームボードの状態に更新する
  const completeBoard = useMemo(
    () =>
      changeBoard()
        .slice(3, 23)
        .map((e) => e.filter((num) => num !== 9)),
    [x, y, rotateNumber]
  )
```
`useCallback`と`KeyboardEvent.code`を使うことでキーボード操作を可能にしました。
```js:index.tsx
// キーボード操作
  const handleKeyDown = useCallback(
    (e: KeyboardEvent) => {
      switch (e.code) {
        case 'ArrowLeft':
          moveLeft() // 左に動く関数
          break
          ~省略~
        case 'ShiftRight':
          holdfunc() // ホールドを行う関数
          break
      }
    },
    [x, y, tetromino, rotateNumber]
  )  
```
https://github.com/Yoshi-program/tetris/blob/main/pages/index.tsx#L469-L514

全体の反省点としてはコードがとても読みづらいことです。特にテトリミノをリセットする関数やホールドを行う関数は変数の更新が多くて、私目線すごく汚く見えました。

# Udemyで学んだこと
マインスイーパーとテトリスを作るまではReact, TypeScriptの具体的なことは学ばず,必要最低限で来ました。そのため、ゲームのソースコードもコンポーネント分割をしておらず、見づらくなっていたと思います。
そのことから、React,TypeScriptについてもっと学ぼうと**Udemy**の講座を受けました。
受けた講座は以下になります
https://www.udemy.com/course/modern_javascipt_react_beginner/
https://www.udemy.com/course/react_stepup/

その中で**Atomic Design**や**React Router**、**カスタムフック**などを学びました。
まだ学びはインプットをした状態で、自分で新しいプロジェクトを作ってアウトプットはあまりできていないため、たくさんコードを書いて自分のものにしたいです。

# やっていないこと
ステート管理(Udemyで少し触れた程度)や環境構築、Dockerのあたりはやっていません。

# やっていきたこと(バイトもしたい)
2年生の講義でC言語を学んでいくので、その技術とReactなどを組み合わせて、何か作っていきたいと考えています。
その中、IT企業でバイトをしてみたいと思っています。大学の講義があるので8月からの夏休みまでは沢山の時間はできないのですが、一般的な大学よりも時間はあると思うので連絡お待ちしております。バイト内容としてはリモートでReact+TSでできて、業務系SaaSの管理画面開発みたいな分野が理想的だと考えています。

# 連絡手段
ここまで読んでいただきありがとうございます！
連絡の際はTwitterにDMを飛ばしていただけると幸いです。
https://twitter.com/YoshiProgrammer

