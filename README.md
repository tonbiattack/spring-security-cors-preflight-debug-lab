# Spring Security CORSプリフライトデバッグラボ

Spring SecurityでCORS統合を有効にしていないため、許可済みオリジンからのプリフライト要求が401になる不具合を再現し、修正するSpring Bootプロジェクトです。

## 対象となる契約

`https://app.example.test` から `GET /api/greeting` を呼び出す前のプリフライトは、認証情報なしで200を返し、許可オリジン・メソッド・ヘッダーを応答します。実際の `GET /api/greeting` は引き続き認証必須です。

| 項目 | 期待値 |
| --- | --- |
| プリフライトのステータス | 200 |
| 許可オリジン | `https://app.example.test` |
| 許可メソッド | `GET` を含む |
| 許可ヘッダー | `Authorization` を含む |
| 認証情報の利用 | `true` |

## 必要な環境

Java 21とMavenが必要です。このプロジェクトはSpring Boot 3.3.12、Spring MVC、Spring Security、MockMvcを使用しています。

## 最新状態の確認

最新のmainブランチでは、`CorsConfigurationSource` とSecurityのCORS統合を設定済みです。

```bash
mvn test
```

プリフライトが200となり、CORSの許可ヘッダーを返すことを確認します。

## バグ状態の再現

不具合状態はGit履歴に残しています。初期コミットをチェックアウトすると、プリフライトが401になりテストが失敗します。

```bash
git checkout 37cce32
mvn test
```

確認後はmainブランチへ戻します。

```bash
git switch main
mvn test
```

## 修正内容

修正前のSecurity設定にはCORS統合がありません。

```java
return http
    .authorizeHttpRequests(authorize -> authorize.anyRequest().authenticated())
    .build();
```

修正後は、具体的な許可オリジン・メソッド・ヘッダーを持つ `CorsConfigurationSource` を定義し、Security設定でCORSを有効にします。

```java
return http
    .cors(Customizer.withDefaults())
    .authorizeHttpRequests(authorize -> authorize.anyRequest().authenticated())
    .build();
```

## 関連資料

調査過程は [docs/debugging-record.md](docs/debugging-record.md)、テスト実行ログは `docs/01-bug-reproduction.log` と `docs/02-fixed-verification.log`、公式資料の確認メモは [docs/references.md](docs/references.md) にあります。
