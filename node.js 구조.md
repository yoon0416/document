## java+spring boot 와 비교한다면 이런식

```
/project-root
│
├── app.js                 ← ★ Node 실행 진입점 (npm start 시 여기부터 시작됨)
│                            → [Spring] Main 클래스 (@SpringBootApplication)

├── routes/
│   └── user.js            ← 요청 경로 등록 (예: /user/login)
│                            → [Spring] @RestController + @RequestMapping("/user")

├── controllers/
│   └── userController.js  ← 요청 처리 로직
│                            → [Spring] UserController.java (@RestController + @PostMapping 등)

├── services/
│   └── userService.js     ← 실제 비즈니스 로직
│                            → [Spring] UserService.java (@Service)

├── models/
│   └── user.js            ← DB 테이블 모델 (Sequelize or Mongoose)
│                            → [Spring] User.java (@Entity + JPA 매핑)

├── config/
│   └── db.js              ← DB 연결 설정
│                            → [Spring] application.yml / application.properties (DB 설정)
│                               + DBConfig.java (DataSource 설정 시)

├── middlewares/
│   └── auth.js            ← JWT, 인증 처리 미들웨어 등
│                            → [Spring] JwtFilter.java + SecurityConfig.java (Spring Security 필터)

├── logs/
│   └── access.log         ← 로그 저장 (블록체인 로그도 여기에 쓸 수 있음)
│                            → [Spring] logback-spring.xml + custom LogUtil.java (로그 기록)

└── ...

```
