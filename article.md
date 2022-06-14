# はじめに

良いコード/悪いコードで学ぶ設計入門（ミノ駆動本）を読んで、これは初心者の必読本だ!と感じ、紹介するために記事を書きました。本の内容の一部である正しいクラスの作り方について具体例を用いて解説します。

初心者： 一度でも一からコードを書いて動くものを作ったことがある方 ~

### 読むべき人

- コードレビューしろと言われても何を基準にレビューをすればいいかわからない方。
- 共通処理を作れと言われても何からどう作ればいいかわからない方。

※ オブジェクト指向を理解している方にとっては当前の内容になります。

### 前提

本記事は JAVA のコードで例を提示していますが、考え方に関する内容なので JAVA を書いたことがなくても問題ありません。

### 身に付くスキル

- 保守や変更がしやすいコードを書くための考え方(クラス設計について)

# 参考資料

[仙塲 大也. 良いコード／悪いコードで学ぶ設計入門保守しやすい　成長し続けるコードの書き方](https://www.amazon.co.jp/dp/B09Y1MWK9N/ref=dp-kindle-redirect?_encoding=UTF8&btkr=1)

# 内容

## クラスの設計方法

筆者が述べていますが、「クラスが単体で正常動作する設計にする」ことはオブジェクト指向を考える上で大切な考え方です。`単体で正常動作`するとはどういうことか、解説していきます。

### 1. データとそのデータの処理をクラス内にまとめよう！

- 👿 [悪いクラスの構造](https://github.com/kdr250/My2DGameSample/commit/6e774a32911db140935317ec55185caa2fb0f94b)
  - クラスのインスタンス変数を操作するロジックが、全く別のクラスに実装されている
    - 悪い理由: 重複コード、修正漏れ、可読性の低下が発生する。

※ hitPointを定義しているクラスは[こちら](https://github.com/kdr250/My2DGameSample/blob/f44863b1a2f2fb10f544d3d4cf33a6c0000c2b7a/src/main/java/entity/Entity.java#L41)
```
public class ItemPotionRed extends Entity {

  public ItemPotionRed(GamePanel gp) {
    super(gp);
    this.gp = gp;

    type = typeConsumable;
    name = "Red Potion";
    recoveryAmount = 5;
    down1 = setup("objects/potion_red", gp.tileSize, gp.tileSize);
    description = "[" + name + "]\nHeals you life by" + recoveryAmount + ".";
  }

  @Override
  public void use(Entity entity) {
    gp.gameState = gp.dialogState;
    gp.ui.currentDialog = "You drink the " + name + "!\nYour life has been recovered by " + recoveryAmount + ".";
    /** 
     * ×リスト2.8 どこかに書かれるヒットポイント回復ロジック
     * (正)Entityクラスに回復する関数を実装する
     */
    entity.hitPoint = entity.hitPoint + recoveryAmount; 
    if (gp.player.hitPoint > gp.player.maxHitPoint) {
      gp.player.hitPoint = gp.player.maxHitPoint;
    }
    gp.playSE(2);
  }

  @Override
  public void setAction() {
  }
}
```
- 👼 良いクラスの構造
  - インスタンス変数に対して操作するメソッドをクラス内に保持している(不正状態から防御)
    - 良い理由: 重複コードが減り(なくしたい)、仕様変更などの処理修正をするのが楽になり、修正漏れも防ぐことができる。
### 2. 初期設定不要なインスタンスを作れるようにしよう！

- 👿 悪いクラスの構造
  - 他のクラスに初期化してもらう必要がある。インスタンスを作成した後に変数に値を入れるする必要があるなど、初期設定が必要な状態。
    - 悪い理由: NPE が発生する可能性がある。
- 👼 良いクラスの構造
  - コンストラクタで値を設定する。
    - 良い理由: 初期化するときに引数に値を入れなければコンパイルエラーが発生する構造になるので NPE が発生しない。

### 3. 変数の値は書き換えられないようにしよう！

👿 [悪いクラスの構造](https://github.com/kdr250/My2DGameSample/commit/de8a29c64be4fcff0e993e5b8a9aedc8148da72e#diff-daa4914f1ad718ed386c63b5fc0c32264e24258f53549797a130f5ac1b142112)

- インスタンス変数が可変(ミュータブル)な状態になっている。
  - 悪い理由: 初期値がわからなくなる。いつどこで変更されたか分かりづらいので、その変数の値の状態を見失い、意図しない値に書き変わってしまう可能性がある。

```
package main.java.objects;

/**
 * ×リスト4.6 攻撃力を表現するクラス
 */
public class AttackPower {
  static final int MIN = 0;
  int value; // finalが付いていないので可変

  public AttackPower(int value) {
    if (value < MIN) {
      throw new IllegalArgumentException();
    }

    this.value = value;
  }

  /**
   * ×リスト4.13 攻撃力を変化させるメソッドを追加
   * 攻撃力を強化する
   * @param increment 攻撃力の増分
   */
  public void reinForce(int increment) {
    value += increment;
  }

  /**
   * ×リスト4.13 攻撃力を変化させるメソッドを追加
   * 無力化する
   */
  public void disable() {
    value = MIN;
  }

  public int value() {
    return value;
  }
}
```

- 👼 良いクラスの構造
  - インスタンス変数が不変(イミュータブル)な状態になっている。値を変更したい場合は変更する値を引数に持った関数を作成し、引数の値をコンストラクタでセットした新たなインスタンスを返り値とする。
    - 良い理由: インスタンス変数の初期状態を保つことができ、副作用を防ぐことができる。


# おわりに
