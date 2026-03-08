# Example: Library Lending Management System / 図書館貸出管理システム

このファイルは `rmodp-workflow-web` の実行サンプルです。
以下の内容をコピーして Claude.ai に貼り付けることで、デモを開始できます。

This file is a sample input for `rmodp-workflow-web`.
Copy the content below and paste it into Claude.ai to start a demo.

---

## 日本語サンプル

```
rmodp-workflow-web 図書館貸出管理システム

【業務概要】
市立図書館の蔵書貸出・返却・予約を管理するシステム。
会員登録した市民が蔵書を借り、期限内に返却する。
延滞が発生した場合は督促を行い、一定期間経過後は貸出停止措置を取る。

【業務シナリオ】
1. 市民が図書館窓口またはオンラインポータルで会員登録を行う。
2. 会員は蔵書検索機能で蔵書を検索し、在庫がある場合は直接借り出す。
   在庫がない場合は予約登録を行い、返却時に通知を受ける。
3. 司書は貸出処理を行い、貸出期限（2週間）を設定する。
4. 会員は期限前に返却するか、最大1回の延長（1週間）を申請できる。
5. 延滞が発生した場合、システムは自動で督促メールを送信する。
6. 延滞が30日を超えた場合、新規貸出・予約を停止する。
7. 司書は蔵書の追加・廃棄・状態管理（修繕中など）を行う。

【業務ルール】
- 会員1人あたりの同時貸出上限は5冊
- 延長は1回のみ可能（延滞中は不可）
- 予約は最大3件まで
- 廃棄処理済みの蔵書は貸出不可
```

---

## English Sample

```
rmodp-workflow-web Library Lending Management System

[Business Overview]
A system managing book lending, returns, and reservations for a public library.
Registered members can borrow books and return them within the due date.
Overdue notices are issued for late returns, and borrowing privileges are suspended after a threshold period.

[Business Scenario]
1. Citizens register as members at the library counter or through the online portal.
2. Members search the catalogue and borrow available books directly.
   If unavailable, they place a reservation and receive notification upon return.
3. Librarians process lending and set a due date (2 weeks).
4. Members return books before the due date or request one extension (1 week).
5. The system automatically sends overdue notices when a due date is passed.
6. After 30 days overdue, new lending and reservations are suspended.
7. Librarians manage the catalogue: adding, decommissioning, and tracking item status (e.g., under repair).

[Business Rules]
- Maximum 5 items on loan per member at any time
- Only one extension per loan (not allowed during overdue status)
- Maximum 3 active reservations per member
- Decommissioned items cannot be lent
```
