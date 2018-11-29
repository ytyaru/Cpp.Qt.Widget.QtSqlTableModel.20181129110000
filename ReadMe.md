# このソフトウェアについて

　Qt5学習。QtでSQLite3を使う。QSqlTableModel.sort()で並び替えした。

* `QSqlTableModel.sort(int fieldIndex, flag)`にて対象レコードを並び替える
* `model.setFilter("");`: フィルタを初期化する

## 前回まで

* https://github.com/ytyaru/Cpp.Qt.Widget.QtSqlTableModel.20181129060000
* https://github.com/ytyaru/Cpp.Qt.Widget.QtSqlTableModel.20181128190000
* https://github.com/ytyaru/Cpp.Qt.Widget.QtSqlTableModel.20181128180000
* https://github.com/ytyaru/Cpp.Qt.Widget.QtSqliteDb.20181128160000
* https://github.com/ytyaru/Cpp.Qt.Widget.QtSqliteDb.20181128120000
* https://github.com/ytyaru/Cpp.Qt.Widget.QSql.SQLite3.Transaction.20181128070000
* https://github.com/ytyaru/Cpp.Qt.Widget.QSql.SQLite3.Class.20181127180000
* https://github.com/ytyaru/Cpp.Qt.Widget.QSql.SQLite3.Class.20181127170000
* https://github.com/ytyaru/Cpp.Qt.Widget.QSql.SQLite3.Class.20181127160000
* https://github.com/ytyaru/Cpp.Qt.Widget.QSql.SQLite3.Class.20181127130000

## コード抜粋

1. DB接続
2. テーブル作成
3. モデル作成
4. モデルのレコードを並び替える
5. 編集の確定
6. select文でレコード確認

## 1. DB接続

```cpp
QSqlDatabase db = QSqlDatabase::database("Memo");
```

## 2. テーブル作成

```cpp
QSqlQuery query(db);
query.exec(tr("create table Memo(id INTEGER PRIMARY KEY AUTOINCREMENT, Memo TEXT, Created TEXT)"));
query.exec(tr("insert into Memo(Memo,Created) values('メモ内容だよ', '1999-12-31 23:59:59')"));
```

## 3. モデル作成

```cpp
QSqlTableModel model(nullptr, db);
model.setTable("Memo");
model.setEditStrategy(QSqlTableModel::EditStrategy::OnManualSubmit); // これがないとmodel.removeRow, removeRowsが反映されない
model.select(); // これがないとquery.exec()で発行したinsert文のデータが残る
```

## 4. モデルのレコードを並び替える

* `model.sort()`で並び替える

```cpp
model.setFilter("");
model.sort(model.record().indexOf("Created"), Qt::SortOrder::DescendingOrder);
```

　並び替えたレコードを参照してデバッグ出力する。

```cpp
for (int r = 0; r < model.rowCount(); r++) {
    QString text;
    for (int c = 0; c < model.columnCount(); c++) {
        text += model.record(r).value(c).toString() + tr("|");
    }
    qDebug() << text;
}
```

　取得された列はQVariant型。これを文字列型に変換した。本番ではint, datetimeなどそれぞれの型に変えるべき。そのコードをどうすれば共通化できるのか気になっている。

## 5. 編集の確定

```cpp
if( model.submitAll() ) {
    model.database().commit();
    qDebug() << "commit()";
} else {
    model.database().rollback();
    qDebug() << "rollback()";
}
```

## 6. select文でレコード確認

```cpp
query.exec(tr("select * from Memo"));
while (query.next()) {
    qDebug() << query.value(0) << "|" << query.value(1) << "|" << query.value(2);
}
```

# 参考

* https://lists.qt-project.org/pipermail/interest/2014-February/011226.html
* https://webcache.googleusercontent.com/search?q=cache:Hc3-4LuQAV4J:https://www.qtcentre.org/threads/27353-delete-row-from-QSqlTableModel-in-QTableView+&cd=1&hl=ja&ct=clnk&gl=jp

## Qt要素

* http://doc.qt.io/qt-5/qsql.html
    * http://doc.qt.io/qt-5/qsqldatabase.html
    * http://doc.qt.io/qt-5/qsqlquery.html
    * http://doc.qt.io/qt-5/qsqlerror.html
    * http://doc.qt.io/qt-5/qsqltablemodel.html
    * http://doc.qt.io/qt-5/qsqlrecord.html
    * http://doc.qt.io/qt-5/qsqlfield.html

# 開発環境

* [Raspberry Pi](https://ja.wikipedia.org/wiki/Raspberry_Pi) 3 Model B+
    * [Raspbian](https://www.raspberrypi.org/downloads/raspbian/) GNU/Linux 9.0 (stretch) 2018-06-27
        * Qt 5.7.1

## 環境構築

* [Raspbian stretch で Qt5.7 インストールしたもの一覧](http://ytyaru.hatenablog.com/entry/2019/12/17/000000)

# ライセンス

　このソフトウェアはCC0ライセンスである。

[![CC0](http://i.creativecommons.org/p/zero/1.0/88x31.png "CC0")](http://creativecommons.org/publicdomain/zero/1.0/deed.ja)

## 利用ライブラリ

ライブラリ|ライセンス|ソースコード
----------|----------|------------
[Qt](http://doc.qt.io/)|[LGPL](http://doc.qt.io/qt-5/licensing.html)|[GitHub](https://github.com/qt)

* [参考1](https://www3.sra.co.jp/qt/licence/index.html)
* [参考2](http://kou-lowenergy.hatenablog.com/entry/2017/02/17/154720)
* [参考3](https://qiita.com/ynuma/items/e8749233677821a81fcc)

