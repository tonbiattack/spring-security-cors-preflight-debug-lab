# 参照資料メモ

## Spring Security Reference

URL: https://docs.spring.io/spring-security/reference/servlet/integrations/cors.html

Spring Securityは、プリフライト要求に認証情報が含まれないため、CORSをSecurityより先に処理する必要があると説明しています。`UrlBasedCorsConfigurationSource` を提供し、Security設定でCORS統合を有効にすると、`CorsFilter` をSecurityと連携できます。

## Spring Framework Reference

URL: https://docs.spring.io/spring-framework/reference/web/webmvc-cors.html

Spring MVCは、明示されたCORS設定がない場合、プリフライト・単純・実際のCORS要求に応答ヘッダーを追加しないと説明しています。プリフライト要求は直接処理され、許可オリジン、メソッド、ヘッダーの設定に基づく応答を返します。認証情報を許可する場合は、許可信頼境界として具体的なオリジンを指定する必要があります。
