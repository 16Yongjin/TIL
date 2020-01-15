# 채용 플랫폼 자동입력

## 사람인 학력사항 자동입력

`userData`에 알맞는 정보를 채우면 된다.

```javascript
const puppeteer = require("puppeteer");

// 입력 필요항목
const userData = {
  id: "", // 아이디
  pw: "", // 비밀번호
  educationLevel: "대학교(4년)", // 학력
  schoolName: "서울대학교", // 학교 이름
  schoolLocation: "서울", // 학교 위치
  majorCategory: "컴퓨터", // 주전공 계열
  majorName: "컴퓨터공학과", // 주전공 이름
  hasMinor: true, // 부/이중/복수 전공 여부 (선택)
  minorType: "이중전공", // 부전공 종류
  minorCategory: "경제/경영", // 부전공 계열
  minorName: "경영학과", // 부전공 이름
  schoolEntrance: "201603", // 입학일자(YYYYMM)
  schoolGraduation: "202003", // 졸업일자(YYYYMM)
  dayOrNight: "주간", // 주간/야간 선택
  gpa: "4.5", // 학점
  gpaScale: "4.5" // 기준학점
};

// 특정 텍스트를 가진 xpath 선택자에 클릭 이벤트를 보내는 함수
const clickWithText = page => async (selector, text) => {
  selector = selector.startsWith("//") ? selector : `//${selector}`;
  await page.waitForXPath(`${selector}[contains(text(), '${text}')]`);
  const [element] = await page.$x(`${selector}[contains(text(), '${text}')]`);

  if (element) {
    console.log(`${selector}[contains(text(), '${text}')]`, "클릭");
    await element.click();
  }

  return null;
};

const main = async () => {
  const browser = await puppeteer.launch({ headless: false });
  const page = await browser.newPage();
  await page.setViewport({ width: 1368, height: 802 });
  await page.goto("https://www.saramin.co.kr");

  // 로그인
  await page.type("#login_person_id", userData.id);
  await page.type("#login_person_pwd", userData.pw);
  await page.click(".btn_login");
  await page.waitForNavigation();

  // 학력사항 입력 페이지로 이동
  await page.goto(
    "https://www.saramin.co.kr/zf_user/member/profile/school-write"
  );

  // 대학교(4년제) 선택
  // [ "학력 선택", "초등학교 졸업", "중학교 졸업", "고등학교 졸업", "대학(2,3년)", "대학교(4년)", "대학원(석사)", "대학원(박사)", "직업전문학원/학교 및 기타학력" ]
  await clickWithText(page)("button", "학력");
  await clickWithText(page)("a", userData.educationLevel);

  // 학교이름 입력 후 자동입력 항목 선택
  await page.type("input[id^=school_nm]", userData.schoolName);
  await page.waitForSelector(".auto_school_name");
  await page.keyboard.press("ArrowDown");
  await page.keyboard.press("Enter");

  // 서울 선택
  // ["서울", "경기", "광주", "대구", "대전", "부산", "울산", "인천", "강원", "경남", "경북", "전남", "전북", "충북", "충남", "제주", "전국", "세종", "아시아·중동", "북·중미", "남미", "유럽", "오세아니아", "아프리카", "남극대륙", "기타해외"]
  await clickWithText(page)("button", "지역 선택");
  await clickWithText(page)("a", userData.schoolLocation);

  // 어문학 선택
  // ["어문학", "영어/영문", "중어/중문", "일어/일문", "국어/국문", "인문과학", "사회과학", "상경계열", "경제/경영", "회계학", "법학계열", "사범계열", "종교학", "생활과학", "예/체능", "자연과학계열", "농수산/해양/축산", "수학/통계학", "물리/천문/기상학", "화학/생물", "공학 계열", "전기/전자/정보통신공학", "컴퓨터/시스템공학", "금속/비금속공학", "생명/화학/환경/바이오", "도시/토목/건축공학", "에너지/원자력 공학", "산업/자동차/우주공학", "기계/조선/항공공학", "신소재/재료/섬유공학", "식품/유전/안전공학", "의학계열", "직접입력"]
  await clickWithText(page)(
    `//div[contains(@class, "area_school_major")]//button`,
    "전공계열 선택"
  );
  await clickWithText(page)(
    `//div[contains(@class, "area_school_major")]//a`,
    userData.majorCategory
  );
  await page.type("input[id^=school_major_15]", userData.majorName);

  // 부/이중/복수전공 입력
  if (userData.hasMinor) {
    await clickWithText(page)("button", "전공 추가하기");
    await page.waitForSelector(".resume_row.area_school_minor");
    await clickWithText(page)("button", "전공구분선택");
    await clickWithText(page)("a", userData.minorType);

    // 이중전공 선택
    await clickWithText(page)(
      `//div[contains(@class, "area_school_minor")]//button`,
      "전공계열 선택"
    );
    await clickWithText(page)(
      `//div[contains(@class, "area_school_minor")]//a`,
      userData.minorCategory
    );
    // 이중전공 입력
    await page.type("input[id^=school_minor_15]", userData.minorName);
  }

  // 입학 & 졸업 일자
  await page.type("input[id^=school_entrance_dt]", userData.schoolEntrance);
  await page.type("input[id^=school_graduation_dt]", userData.schoolGraduation);

  // 주/야간 선택
  await clickWithText(page)("button", "주/야간 선택");
  await clickWithText(page)("a", userData.dayOrNight);

  // 학점 입력
  await page.type("input[id^=school_major_avg]", userData.gpa);
  await clickWithText(page)("button", "기준학점선택");
  await clickWithText(page)(
    '//div[contains(@class, "area_grades")]//a',
    userData.gpaScale
  );

  // 작성완료 버튼
  await clickWithText(page)("button", "작성완료");
};

main();
```
