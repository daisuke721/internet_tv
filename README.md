## インターネットTVみたいなデータベースを構築しよう！
#### はじめに
好きな時間に好きな場所で話題の動画を無料で楽しめる「インターネットTVサービス」を作成することになりました。
データベース設計をした上でデータを取得するSQLを作成しよう！

---

#### 仕様
* ドラマ1、ドラマ2、アニメ1、アニメ2といった複数のチャンネルがある
* 各チャンネルの下では時間帯ごとに番組枠が1つ設定されており、番組が放映される
* 各番組はシリーズになっており単発ものがある。シリーズものはシーズン1、シーズン2のように複数あり、各シーズンの下では各エピソードが設定されている
* 再放送もあるため、ある番組が複数チャンネルの異なる番組枠で放映される
* 番組の情報として、タイトル、番組詳細、ジャンルが画面上に表示される
* 各エピソードの情報として、シーズン数、エピソード数、タイトル、エピソード詳細、動画時間、公開日、視聴数が画面上に表示される。単発のエピソードの場合はシーズン数、エピソード数は表示されない。
* ジャンルとしてアニメ、映画、ドラマ、ニュースなどがある。
* KPIとして、チャンネルの番組枠のエピソードごとに視聴数を記録する。なお、一つのエピソードは複数の異なるチャンネル及び番組枠で放送されることがあるので、属するチャンネルの番組枠ごとに視聴数がどうだったかも追えるようにする。

---

#### ER図
![ER図](/image/Untitled.png)

---

#### 1. MySQLにログイン
```bash
# user_nameの部分は自分で作成したユーザでログイン
mysql -u user_name -p
```

---

#### 2. データベースの作成
```bash
# internet_tvという名前のデータベースを作成
CREATE DATABASE internet_tv;
```
* データベースを作成したら`SHOW DATABASES;`コマンドで確認！！

---

#### 3. テーブルの作成
下記の各テーブルのコマンドを使用し一つずつ実行しテーブルを作成
テーブルを作成したら`SHOW tables;`で確認！！
また、`DESC テーブル名;`で各テーブルの構造も確認できる

* **channelsテーブル**

| カラム名      | データ型        | NULL | キー     | 初期値              | AUTO INCREMENT |
|---------------|-----------------|------|----------|---------------------|----------------|
| id            | INT             |      | PRIMARY  |                     | YES            |
| title         | VARCHAR(100)    |      |          |                     |                |
| start_time    | DATETIME        |      |          |                     |                |
| end_time      | DATETIME        |      |          |                     |                |
| created_at    | TIMESTAMP       |      |          | CURRENT_TIMESTAMP   |                |
| updated_at    | TIMESTAMP       | YES  |          | CURRENT_TIMESTAMP   |                |

```bash
CREATE TABLE channels (
  id INT PRIMARY KEY AUTO_INCREMENT,
  title VARCHAR(100) NOT NULL,
  start_time DATETIME NOT NULL,
  end_time DATETIME NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

* **programsテーブル**

| カラム名      | データ型        | NULL | キー             | 初期値            | AUTO INCREMENT |
|---------------|-----------------|------|------------------|-------------------|----------------|
| id            | INT             |      | PRIMARY         |                   | YES            |
| channel_id    | INT             | YES  | FOREIGN, INDEX  |                   |                |
| title         | VARCHAR(100)    |      |                 |                   |                |
| start_time    | DATETIME        |      |                 |                   |                |
| end_time      | DATETIME        |      |                 |                   |                |
| detail        | TEXT            |      |                 |                   |                |
| created_at    | TIMESTAMP       |      |                 | CURRENT_TIMESTAMP |                |
| updated_at    | TIMESTAMP       | YES  |                 | CURRENT_TIMESTAMP |                |

```bash
CREATE TABLE programs (
  id INT PRIMARY KEY AUTO_INCREMENT,
  channel_id INT,
  title VARCHAR(100) NOT NULL,
  start_time DATETIME NOT NULL,
  end_time DATETIME NOT NULL,
  detail TEXT NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (channel_id) REFERENCES channels(id),
  INDEX (channel_id)
);
```

* **genresテーブル**

| カラム名      | データ型        | NULL | キー     | 初期値            | AUTO INCREMENT |
|---------------|-----------------|------|----------|-------------------|----------------|
| id            | INT             |      | PRIMARY  |                   | YES            |
| channel_id    | INT             | YES  | FOREIGN  |                   |                |
| program_id    | INT             | YES  | FOREIGN  |                   |                |
| title         | VARCHAR(100)    |      |          |                   |                |
| created_at    | TIMESTAMP       |      |          | CURRENT_TIMESTAMP |                |
| updated_at    | TIMESTAMP       | YES  |          | CURRENT_TIMESTAMP |                |

```bash
CREATE TABLE genres (
  id INT PRIMARY KEY AUTO_INCREMENT,
  channel_id INT,
  program_id INT,
  title VARCHAR(100) NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (channel_id) REFERENCES channels(id),
  FOREIGN KEY (program_id) REFERENCES programs(id)
);
```

* **seasonsテーブル**

| カラム名      | データ型        | NULL | キー     | 初期値            | AUTO INCREMENT |
|---------------|-----------------|------|----------|-------------------|----------------|
| id            | INT             |      | PRIMARY  |                   | YES            |
| program_id    | INT             | YES  | FOREIGN  |                   |                |
| title         | VARCHAR(100)    |      |          |                   |                |
| created_at    | TIMESTAMP       |      |          | CURRENT_TIMESTAMP |                |
| updated_at    | TIMESTAMP       | YES  |          | CURRENT_TIMESTAMP |                |

```bash
CREATE TABLE seasons (
  id INT PRIMARY KEY AUTO_INCREMENT,
  program_id INT,
  title VARCHAR(100) NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (program_id) REFERENCES programs(id)
);
```

* **episodesデーブル**

| カラム名           | データ型        | NULL | キー             | 初期値            | AUTO INCREMENT |
|--------------------|-----------------|------|------------------|-------------------|----------------|
| id                 | INT             |      | PRIMARY         |                   | YES            |
| program_id         | INT             | YES  | FOREIGN, INDEX  |                   |                |
| season_id          | INT             | YES  | FOREIGN         |                   |                |
| title              | VARCHAR(100)    |      |                 |                   |                |
| detail             | TEXT            |      |                 |                   |                |
| video_time         | TIME            |      |                 |                   |                |
| publication_date   | DATETIME        |      |                 |                   |                |
| view               | BIGINT          |      |                 |                   |                |
| created_at         | TIMESTAMP       |      |                 | CURRENT_TIMESTAMP |                |
| updated_at         | TIMESTAMP       | YES  |                 | CURRENT_TIMESTAMP |                |

```bash
CREATE TABLE episodes (
  id INT PRIMARY KEY AUTO_INCREMENT,
  program_id INT,
  season_id INT,
  title VARCHAR(100) NOT NULL,
  detail TEXT NOT NULL,
  video_time TIME NOT NULL,
  publication_date DATETIME NOT NULL,
  view BIGINT NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (program_id) REFERENCES programs(id),
  FOREIGN KEY (season_id) REFERENCES seasons(id),
  INDEX (program_id)
);
```

* **rankingsテーブル**

| カラム名      | データ型        | NULL | キー     | 初期値            | AUTO INCREMENT |
|---------------|-----------------|------|----------|-------------------|----------------|
| id            | INT             |      | PRIMARY  |                   | YES            |
| program_id    | INT             | YES  | FOREIGN  |                   |                |
| ranking       | INT             |      |          |                   |                |
| created_at    | TIMESTAMP       |      |          | CURRENT_TIMESTAMP |                |
| updated_at    | TIMESTAMP       | YES  |          | CURRENT_TIMESTAMP |                |

```bash
CREATE TABLE rankings (
  id INT PRIMARY KEY AUTO_INCREMENT,
  program_id INT,
  ranking INT NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (program_id) REFERENCES programs(id)
);
```

---

#### 4. データの挿入
予め作成したサンプルデータをsample_dataディレクトリのデータファイルから順番にコマンド入力しよう
```bash
1. mysql -u user_name -p internet_tv < sample_data/sample_channels.sql
2. mysql -u user_name -p internet_tv < sample_data/sample_programs.sql
3. mysql -u user_name -p internet_tv < sample_data/sample_genres.sql
4. mysql -u user_name -p internet_tv < sample_data/sample_seasons.sql
5. mysql -u user_name -p internet_tv < sample_data/sample_episodes.sql
6. mysql -u user_name -p internet_tv < sample_data/sample_rankings.sql
```
---

#### 5. クエリを書いてみよう

1. よく見られているエピソードを知りたいです。エピソード視聴数トップ3のエピソードタイトルと視聴数を取得してください
```bash
SELECT
  title, view
FROM
  episodes
ORDER
  BY view DESC
LIMIT 3;
```

**出力結果**

| title                       | view |
|----------------------------|-------|
| エピソード1 - 朝のニュース速報 | 300000 |
| エピソード2 - 交通情報       | 290000 |
| エピソード3 - 天気予報       | 280000 |

---

2. よく見られているエピソードの番組情報やシーズン情報も合わせて知りたいです。エピソード視聴数トップ3の番組タイトル、シーズン数、エピソード数、エピソードタイトル、視聴数を取得してください
```bash
SELECT
  programs.title AS program_title,
  seasons.title AS season_title,
  episodes.title AS episode_title,
  episodes.view AS view_count
FROM
  episodes
JOIN
  programs ON episodes.program_id = programs.id
JOIN
  seasons ON episodes.season_id = seasons.id
ORDER BY
  episodes.view DESC
LIMIT 3;
```

**出力結果**

| program_title     | season_title | episode_title  |  view_count |
|-------------------|-------------|-----------------------------|--------|
| スポーツニュース2    |              | エピソード1 - 朝のニュース速報 | 300000 |
| スポーツニュース2    |              | エピソード2 - 交通情報        | 290000 |
| スポーツニュース2    |              | エピソード3 - 天気予報        | 280000 |

---

3. 本日の番組表を表示するために、本日、どのチャンネルの、何時から、何の番組が放送されるのかを知りたいです。本日放送される全ての番組に対して、チャンネル名、放送開始時刻(日付+時間)、放送終了時刻、シーズン数、エピソード数、エピソードタイトル、エピソード詳細を取得してください。なお、番組の開始時刻が本日のものを本日方法される番組とみなすものとします
```bash
SELECT
  channels.title AS channel_name,
  programs.start_time AS program_start_time,
  programs.end_time AS program_end_time,
  seasons.title AS season_title,
  episodes.title AS episode_title,
  episodes.detail AS episode_detail,
  episodes.view AS view_count
FROM
  programs
JOIN
  channels ON programs.channel_id = channels.id
JOIN
  seasons ON seasons.program_id = programs.id
JOIN
  episodes ON episodes.season_id = seasons.id
WHERE
  DATE(programs.start_time) = CURRENT_DATE
ORDER BY
  programs.start_time;
```

**出力結果**

| channel_name | program_start_time  | program_end_time    | season_title  | episode_title               | episode_detail                                  | view_count |
|--------------|---------------------|---------------------|---------------|-----------------------------|-------------------------------------------------|------------|
| アニメ1      | 2024-11-03 06:00:00 | 2024-11-03 07:00:00 |               | エピソード1 - アニメ名作特集  | アニメの名作を深掘り。                          |     100000 |
| アニメ1      | 2024-11-03 06:00:00 | 2024-11-03 07:00:00 |               | エピソード2 - キャラクター紹介 | 人気キャラクターについて紹介。                  |      95000 |
| アニメ1      | 2024-11-03 06:00:00 | 2024-11-03 07:00:00 |               | エピソード3 - ストーリーの魅力 | 名作のストーリーを分析。                        |      98000 |
| アニメ1      | 2024-11-03 06:00:00 | 2024-11-03 07:00:00 |               | エピソード4 - 名場面集         | 心に残る名場面を特集。                          |      99000 |
| アニメ1      | 2024-11-03 06:00:00 | 2024-11-03 07:00:00 |               | エピソード5 - 制作秘話         | 制作の裏側を公開。                              |      92000 |
| アニメ1      | 2024-11-03 07:00:00 | 2024-11-03 08:00:00 |               | エピソード1 - 最新アニメ紹介   | 最新のアニメを紹介。                            |      95000 |
| アニメ1      | 2024-11-03 07:00:00 | 2024-11-03 08:00:00 |               | エピソード2 - ファンタジー特集 | ファンタジージャンルの作品を特集。              |      93000 |
| アニメ1      | 2024-11-03 07:00:00 | 2024-11-03 08:00:00 |               | エピソード3 - SFアニメの世界   | SFアニメの未来を描く。                          |      92000 |
| アニメ1      | 2024-11-03 07:00:00 | 2024-11-03 08:00:00 |               | エピソード4 - ミステリーとサスペンス | 推理系アニメを紹介。                          |      94000 |
| アニメ1      | 2024-11-03 07:00:00 | 2024-11-03 08:00:00 |               | エピソード5 - おすすめアニメ   | 今注目のアニメを紹介。                          |      91000 |
| アニメ2      | 2024-11-03 08:00:00 | 2024-11-03 09:00:00 | シーズン1     | エピソード1 - 話題のアニメ     | 話題になっているアニメを解説。                  |      90000 |
| アニメ2      | 2024-11-03 08:00:00 | 2024-11-03 09:00:00 | シーズン1     | エピソード2 - 異世界アニメの魅力 | 異世界アニメの見どころを解説。                |      88000 |
| アニメ2      | 2024-11-03 08:00:00 | 2024-11-03 09:00:00 | シーズン1     | エピソード3 - スポーツアニメ特集 | スポーツアニメの人気シーン。                  |      87000 |
| アニメ2      | 2024-11-03 08:00:00 | 2024-11-03 09:00:00 | シーズン1     | エピソード4 - アクションと冒険 | アクションアニメの名場面。                      |      86000 |
| アニメ2      | 2024-11-03 08:00:00 | 2024-11-03 09:00:00 | シーズン1     | エピソード5 - 感動シーン       | 泣けるシーンを特集。                            |      85000 |
| アニメ2      | 2024-11-03 08:00:00 | 2024-11-03 09:00:00 | シーズン2     | シーズン1 エピソード1 - ドラマの舞台裏 | ドラマの背景やキャストについて。            |      80000 |
| アニメ2      | 2024-11-03 08:00:00 | 2024-11-03 09:00:00 | シーズン2     | シーズン1 エピソード2 - 制作秘話 | ドラマ制作の秘話を紹介。                      |      78000 |
| アニメ2      | 2024-11-03 08:00:00 | 2024-11-03 09:00:00 | シーズン2     | シーズン1 エピソード3 - シーン解説 | 重要なシーンの解説。                          |      76000 |
| アニメ2      | 2024-11-03 08:00:00 | 2024-11-03 09:00:00 | シーズン2     | シーズン1 エピソード4 - 視聴者の反応 | 視聴者の感想を紹介。                          |      74000 |
| アニメ2      | 2024-11-03 08:00:00 | 2024-11-03 09:00:00 | シーズン2     | シーズン1 エピソード5 - クライマックス | 物語が最高潮に達する。                      |      77000 |

---

4. ドラマというチャンネルがあったとして、ドラマのチャンネルの番組表を表示するために、本日から一週間分、何日の何時から何の番組が放送されるのかを知りたいです。ドラマのチャンネルに対して、放送開始時刻、放送終了時刻、シーズン数、エピソード数、エピソードタイトル、エピソード詳細を本日から一週間分取得してください
```bash
SELECT
  programs.start_time AS program_start_time,
  programs.end_time AS program_end_time,
  seasons.title AS season_title,
  episodes.title AS episode_title,
  episodes.detail AS episode_detail,
  episodes.view AS view_count
FROM
  programs
JOIN
  channels ON programs.channel_id = channels.id
JOIN
  seasons ON seasons.program_id = programs.id
JOIN
  episodes ON episodes.season_id = seasons.id
WHERE
  channels.title IN ('ドラマ1', 'ドラマ2')
  AND DATE(programs.start_time) BETWEEN CURRENT_DATE AND DATE_ADD(CURRENT_DATE, INTERVAL 7 DAY)
ORDER BY
  programs.start_time;
```

**出力結果**

| program_start_time  | program_end_time    | season_title  | episode_title                       | episode_detail                                | view_count |
|---------------------|---------------------|---------------|-------------------------------------|-----------------------------------------------|------------|
| 2024-11-04 09:00:00 | 2024-11-04 10:00:00 | シーズン1     | シーズン2 エピソード1 - 新たな展開   | 新シーズンの展開について。                    |      77000 |
| 2024-11-04 09:00:00 | 2024-11-04 10:00:00 | シーズン1     | シーズン2 エピソード2 - キャラクターの成長 | 登場人物の成長を描く。                  |      76000 |
| 2024-11-04 09:00:00 | 2024-11-04 10:00:00 | シーズン1     | シーズン2 エピソード3 - 過去との対決 | 主人公が過去と向き合う。                      |      75000 |
| 2024-11-04 09:00:00 | 2024-11-04 10:00:00 | シーズン1     | シーズン2 エピソード4 - 衝撃の真実   | 重要な秘密が明らかに。                        |      74000 |
| 2024-11-04 09:00:00 | 2024-11-04 10:00:00 | シーズン1     | シーズン2 エピソード5 - 最終決戦     | 物語の最終章。                                |      80000 |
| 2024-11-04 09:00:00 | 2024-11-04 10:00:00 | シーズン2     | シーズン1 エピソード1 - 新作の見どころ | 新ドラマの見どころを紹介。                  |      85000 |
| 2024-11-04 09:00:00 | 2024-11-04 10:00:00 | シーズン2     | シーズン1 エピソード2 - キャストインタビュー | キャストインタビューをお届け。          |      83000 |
| 2024-11-04 09:00:00 | 2024-11-04 10:00:00 | シーズン2     | シーズン1 エピソード3 - 名シーン     | 印象に残るシーンを特集。                      |      82000 |
| 2024-11-04 09:00:00 | 2024-11-04 10:00:00 | シーズン3     | シーズン2 エピソード1 - 新展開の始まり | 新たな展開の幕開け。                        |      81000 |
| 2024-11-04 09:00:00 | 2024-11-04 10:00:00 | シーズン3     | シーズン2 エピソード2 - 友情と信頼   | 登場人物の友情が試される。                    |      80000 |
| 2024-11-04 09:00:00 | 2024-11-04 10:00:00 | シーズン3     | シーズン2 エピソード3 - クライマックスへの準備 | クライマックスに向けて盛り上がる。    |      79000 |
| 2024-11-04 11:00:00 | 2024-11-04 12:00:00 |               | シーズン3 エピソード1 - 謎が深まる   | 物語の謎が更に深まる。                        |      78000 |
| 2024-11-04 11:00:00 | 2024-11-04 12:00:00 |               | シーズン3 エピソード2 - 最後の戦い   | 最終章に向けた戦いが始まる。                  |      77000 |
| 2024-11-04 11:00:00 | 2024-11-04 12:00:00 |               | シーズン3 エピソード3 - 物語の結末   | 感動の結末。                                  |      76000 |
