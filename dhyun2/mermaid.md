graph TD
A[웹 보안] --> B[능동적 공격]
A --> C[수동적 공격]

    B --> B1[SQL 인젝션]
    B --> B2[OS 명령 인젝션]

    C --> C1[XSS]
    C --> C2[CSRF]
    C --> C3[클릭재킹]

    C1 --> C1_1[반사형 XSS]
    C1 --> C1_2[저장형 XSS]
    C1 --> C1_3[DOM 기반 XSS]

    A --> D[HTTP 보안]
    D --> D1[HTTPS/TLS]
    D1 --> D2[HSTS]
    D2 --> D3[보안 헤더]

    D3 --> D3_1[CSP]
    D3 --> D3_2[CORS 헤더]

    A --> E[접근 제어]
    E --> E1[동일 출처 정책]
    E1 --> E2[CORS]
    E2 --> E3[PostMessage]

    C1 --> F[XSS 방어]
    F --> F1[입력 검증]
    F --> F2[출력 이스케이프]
    F --> F3[CSP 적용]

    D3_1 --> F3
    E2 --> D3_2
