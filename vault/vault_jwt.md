## Vault JWT auth steps

References from https://www.vaultproject.io/docs/auth/jwt/#jwt-authentication

Enable Vault backend
```
vault auth enable jwt
```

Create private and public keys using openssl 

```
openssl genrsa -out private.pem 2048
openssl rsa -in private.pem -outform PEM -pubout -out pubkey.pem
```

Configure JWT in vault
```
vault write auth/jwt/config jwt_validation_pubkeys=@pubkey.pem
```

Create vault role with policies attached
```
vault write auth/jwt/role/demo policies=default user_claim='sub' role_type='jwt' groups_claim='groups' bound_audiences="http://127.0.0.1:8200" bound_subject=""
```

generate JWT 
```
https://jwt.io/#debugger-io?token=eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoiSm9obiBEb2UiLCJzdWIiOiJ0ZXN0IiwiYWRtaW4iOnRydWUsImdyb3VwcyI6WyIiXSwiYXVkIjoiaHR0cDovLzEyNy4wLjAuMTo4MjAwIiwibmJmIjoxNDY4NDEwNTc4LCJleHAiOjE2Njg0MjA1Nzh9.qbBJXU33JcZyUjmBCJzIRnY6_0E2tGPNRniVompoWVJDUTvThML9Zu8wDVy9lxd58_TcwxH7vp76yRVk3DL1A5qVKXqJIBS0ScnOUb0wYbCsH60uHQ_3kdqJa-gvPQcnN6PNY8FhngTqS7xtN7q2A36d7RVkzZBoygXGj8WrvSL0p4QZMuLOhHAqSD8r2IXgnyFYCtKjK9cYU6Q45fWoZzemmC16DxRiMW5fiM0eoPBwjJxcuM7CvU-t3GVH-QSwOwa9ZYrdGGSqTN40XXiGX0TCLhB9yzpLBhb9c3XVtZvE5c4z-pQOrVnxrcR-GmrszGzpsgtO8d97BiJQLxXMhg&publicKey=-----BEGIN%20PUBLIC%20KEY-----%0AMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAxLsGElDMm2ghDNfa377K%0AbskQGRzyFXu43QLG1%2BRiq7pVpNYMfRfW2yQHotypI73qBQYTT5QQHlRoWT7ipFs0%0AVgTsq8eHgQqjNJ7m6azA8PPiLh%2B2OFLrtgQIr7%2Bm2P2RDXuJLLH%2FRRfpo%2FbOMQFk%0AFagdfE4B%2FyWkYi5XOdvUEOm2vhdQ89Ed%2Fsu3xHlpUeV4n150FU5E1%2Fd%2FLMrR0lQz%0AABtVuLsVP8p%2BY74qQQH6pvBp3IFuhNGx%2FDfXiuUJxBCOzPTa4dtOeWhl%2F886PSLZ%0A1PcluDIyhU549Yzo2mOUlZ7NUQsgfHTVtHbsN%2Fpvf96p5cfSJAj0exxz104oruZ0%0ARwIDAQAB%0A-----END%20PUBLIC%20KEY-----
```

test vault login using JWT keys
```
vault write auth/jwt/login role=demo jwt=eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoiSm9obiBEb2UiLCJzdWIiOiJ0ZXN0IiwiYWRtaW4iOnRydWUsImdyb3VwcyI6WyIiXSwiYXVkIjoiaHR0cDovLzEyNy4wLjAuMTo4MjAwIiwibmJmIjoxNDY4NDEwNTc4LCJleHAiOjE2Njg0MjA1Nzh9.qbBJXU33JcZyUjmBCJzIRnY6_0E2tGPNRniVompoWVJDUTvThML9Zu8wDVy9lxd58_TcwxH7vp76yRVk3DL1A5qVKXqJIBS0ScnOUb0wYbCsH60uHQ_3kdqJa-gvPQcnN6PNY8FhngTqS7xtN7q2A36d7RVkzZBoygXGj8WrvSL0p4QZMuLOhHAqSD8r2IXgnyFYCtKjK9cYU6Q45fWoZzemmC16DxRiMW5fiM0eoPBwjJxcuM7CvU-t3GVH-QSwOwa9ZYrdGGSqTN40XXiGX0TCLhB9yzpLBhb9c3XVtZvE5c4z-pQOrVnxrcR-GmrszGzpsgtO8d97BiJQLxXMhg
```


