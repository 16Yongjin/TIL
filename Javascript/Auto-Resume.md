# 채용 플랫폼 로그인

```javascript
const puppeteer = require("puppeteer");

const main = async () => {
  const browser = await puppeteer.launch({ headless: false });
  const page = await browser.newPage();
  await page.goto("https://www.saramin.co.kr");

  // 로그인
  await page.type("#login_person_id", saramin_id);
  await page.type("#login_person_pwd", saramin_pw);
  await page.click(".btn_login");
  await page.waitForNavigation();

  // 내 프로필로 이동
  await page.goto(
    "https://www.saramin.co.kr/zf_user/member/profile/school-write"
  );
  await page.waitForNavigation();
};

main();
```
