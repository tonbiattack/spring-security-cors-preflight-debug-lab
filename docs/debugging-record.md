# CORSプリフライト401のデバッグ記録

## 期待する契約

許可済みオリジン `https://app.example.test` から `GET /api/greeting` を呼ぶためのプリフライトは、認証情報なしで200を返し、許可オリジン、メソッド、ヘッダーを含む応答を返します。

| 項目 | 期待値 |
| --- | --- |
| HTTPステータス | 200 |
| `Access-Control-Allow-Origin` | `https://app.example.test` |
| `Access-Control-Allow-Methods` | `GET` を含む |
| `Access-Control-Allow-Headers` | `Authorization` を含む |
| `Access-Control-Allow-Credentials` | `true` |

## バグ状態での観測

初期コミットで `mvn test` を実行しました。実行ログは [01-bug-reproduction.log](01-bug-reproduction.log) です。

| 観測項目 | 実測値 | 判断 |
| --- | --- | --- |
| プリフライトのHTTPステータス | 401 | CORS応答の前に認証で拒否されている |
| `WWW-Authenticate` | Basic realm | Basic認証エントリーポイントが応答している |
| `Access-Control-Allow-Origin` | なし | CORSポリシーが応答されていない |
| `GreetingController` | 未到達 | コントローラー処理の問題ではない |
| Securityログ | `Securing OPTIONS /api/greeting` | `OPTIONS` がSecurityの認可まで進んでいる |

Securityログには匿名認証の後、Basic認証のエントリーポイントが選択されたことが残っています。プリフライトはユーザーの認証を行う要求ではないため、CORS処理が先に行われるべきです。

## 原因

SecurityフィルターチェーンでCORS統合を有効にしておらず、`CorsConfigurationSource` も定義していませんでした。そのため、`OPTIONS /api/greeting` は通常の認証必須要求として `anyRequest().authenticated()` まで進み、401になりました。

## 修正

具体的な許可オリジン、メソッド、ヘッダーを持つ `CorsConfigurationSource` を追加し、Security設定で `.cors(Customizer.withDefaults())` を有効にしました。

```java
configuration.setAllowedOrigins(List.of("https://app.example.test"));
configuration.setAllowedMethods(List.of("GET"));
configuration.setAllowedHeaders(List.of("Authorization", "Content-Type"));
configuration.setAllowCredentials(true);
```

修正後のSecurityフィルターチェーンには `CorsFilter` が含まれます。プリフライトはこのフィルターで処理されるため、認証エントリーポイントに到達せず、許可ヘッダーを含む200を返します。

## 回帰検証

統合テストは、プリフライトの200だけでなく、許可オリジン、認証情報、許可メソッド、許可ヘッダーを検証します。これにより、API全体を匿名アクセスにして200を返すだけの修正を防ぎ、CORSポリシーの範囲を固定します。
