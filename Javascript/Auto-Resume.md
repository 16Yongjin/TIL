# 채용 플랫폼 로그인

```javascript
const puppeteer = require("puppeteer");

const main = async () => {
  const browser = await puppeteer.launch({ headless: false });
  const page = await browser.newPage();
  await page.goto("https://www.saramin.co.kr");

  await page.evaluate(
    (id, pw) => {
      document.querySelector("#login_person_id").value = id;
      document.querySelector("#login_person_pwd").value = pw;
    },
    saramin_id,
    saramin_pw
  );
  await page.click(".btn_login");
  await page.waitForNavigation();
};

main();
```
