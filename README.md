# Лаборатори №5 — API систем тест: Postman ба Newman

**Хичээл:** F.CSA313 — Программ хангамжийн чанарын баталгаа ба тест  
**Оюутан:** Ө. Анар  
**Оюутны код:** B242270129  

---

## 1. Зорилго

Энэхүү лабораторийн ажлаар `POST /registrations` REST API функцийг тест дизайны 5 алхмын дагуу шинжилж, Postman collection үүсгэн, Newman ашиглан командын мөрөөс автомат тест ажиллуулав. Тестүүдийг бие даасан байхаар setup хүсэлтүүдтэй зохион байгуулж, амжилттай, алдаатай, хязгаарын болон давхар алдааны нөхцлүүдийг шалгав.

---

## 2. Ашигласан орчин

```text
OS: Debian
Node.js: v20.19.2
Newman: 6.2.2
Postman Collection schema: v2.1.0
Base URL: http://localhost:3000
```

Collection variable:

```text
{{baseUrl}} = http://localhost:3000
```

---

## 3. Тестлэгдсэн үндсэн функц

```http
POST /registrations
Content-Type: application/json
```

Жишээ хүсэлт:

```json
{
  "studentID": "B231234567",
  "courseID": "CS313"
}
```

Амжилттай үед:

```json
{
  "result": "OK",
  "registrationID": 1
}
```

`registrationID` нь сервер ажиллаж байх хугацаанд өсдөг тул тестэнд яг тодорхой ID утга шаардаагүй.

---

## 4. Сонголт ба төлөөлөх утгууд

| Сонголт | Төлөөлөх утгууд |
|---|---|
| Оюутны төлөв | active, inactive, байхгүй |
| Оюутны үзсэн хичээл | prerequisite хангана, хангахгүй, хоосон `[]` |
| Хичээлийн төлөв | байгаа, байхгүй |
| Хичээлийн prerequisite | бүгд хангагдсан, хангагдаагүй, prerequisite байхгүй |
| `studentID` талбар | зөв утгатай, байхгүй |
| `courseID` талбар | зөв утгатай, байхгүй |
| JSON формат | зөв JSON, malformed JSON |
| Давхар алдаа | student + course байхгүй, inactive + prerequisite дутуу |

---

## 5. Тестийн спецификаци

| ID | Тестийн нөхцөл | Хүлээгдэх HTTP status | Хүлээгдэх `result` |
|---|---|---:|---|
| TC01 | Active оюутан, prerequisite хангагдсан, course байгаа | 201 | `OK` |
| TC02 | Оюутан байхгүй | 200 | `ERROR_NO_STUDENT` |
| TC03 | Оюутан inactive | 200 | `ERROR_INACTIVE_STUDENT` |
| TC04 | Course байхгүй | 200 | `ERROR_NO_COURSE` |
| TC05 | Active оюутан боловч prerequisite хангаагүй | 200 | `ERROR_PREREQUISITES` |
| TC06 | Prerequisite байхгүй course, `coursesTaken=[]` | 201 | `OK` |
| TC07 | `studentID` талбар дутуу | 400 | `ERROR_BAD_REQUEST` |
| TC08 | `courseID` талбар дутуу | 400 | `ERROR_BAD_REQUEST` |
| TC09 | Malformed JSON | 400 | `ERROR_BAD_JSON` |
| TC10 | Оюутан болон course хоёулаа байхгүй | 200 | `ERROR_NO_STUDENT` |
| TC11 | Оюутан inactive бөгөөд prerequisite мөн хангаагүй | 200 | `ERROR_INACTIVE_STUDENT` |

---

## 6. Давхар алдааны туршилтаар тогтоосон дараалал

API-ийн зааварт нэг хүсэлтэд хэд хэдэн алдаа зэрэг тохиолдвол аль алдааг эхэлж буцаахыг тодорхойлоогүй байсан. Иймээс үүнийг тестээр тогтоов.

| Тохиолдол | Бодит үр дүн | Дүгнэлт |
|---|---|---|
| Student байхгүй + Course байхгүй | `ERROR_NO_STUDENT` | Student existence шалгалт түрүүлж хийгдэж байна |
| Student inactive + prerequisite дутуу | `ERROR_INACTIVE_STUDENT` | Student status шалгалт prerequisite шалгалтаас түрүүлж байна |

---

## 7. Postman collection

Үндсэн collection:

```text
lab05-collection.json
```

Зориуд алдаатай oracle бүхий collection:

```text
lab05-collection-fail.json
```

Collection дотор нийт:

```text
Requests: 25
Test scripts: 11
Assertions: 25
```

Setup хийхэд `PUT /students/:id`, `PUT /courses/:id` хүсэлтүүд ашигласан. Үндсэн test case бүр `POST /registrations` хүсэлт болон status/result oracle-той.

---

## 8. Newman автомат тестийн үр дүн

### 8.1 PASS ажиллуулалт

Команд:

```bash
newman run lab05-collection.json 2>&1 | tee results/newman-pass.txt
echo "exit=${PIPESTATUS[0]}"
```

Үр дүн:

| Үзүүлэлт | Executed | Failed |
|---|---:|---:|
| Iterations | 1 | 0 |
| Requests | 25 | 0 |
| Test scripts | 11 | 0 |
| Assertions | 25 | 0 |

```text
Exit code: 0
Assertions executed: 25
Assertions failed: 0
```

Бүтэн Newman гаралт:

```text
results/newman-pass.txt
```

---

### 8.2 Зориудын FAIL ажиллуулалт

`lab05-collection-fail.json` файлд TC01-ийн амжилттай бүртгэлийн бодит статус `201` байхад oracle-ийг зориуд `200` гэж хүлээдэг болгосон.

Команд:

```bash
newman run lab05-collection-fail.json 2>&1 | tee results/newman-fail.txt
echo "exit=${PIPESTATUS[0]}"
```

Үр дүн:

| Үзүүлэлт | Executed | Failed |
|---|---:|---:|
| Iterations | 1 | 0 |
| Requests | 25 | 0 |
| Test scripts | 11 | 0 |
| Assertions | 25 | 1 |

```text
Failure:
Status code is 201
expected response to have status code 200 but got 201

Exit code: 1
Assertions executed: 25
Assertions failed: 1
```

Бүтэн Newman гаралт:

```text
results/newman-fail.txt
```

Энэ нь API сервер ажиллаж байсан боловч oracle зориуд буруу байсан **assertion failure** юм.

---

### 8.3 SERVER DOWN ажиллуулалт

Серверийг `Ctrl+C` ашиглан зогсоосны дараа зөв үндсэн collection-ийг ажиллуулав.

Команд:

```bash
newman run lab05-collection.json 2>&1 | tee results/newman-down.txt
echo "exit=${PIPESTATUS[0]}"
```

Үр дүн:

| Үзүүлэлт | Executed | Failed |
|---|---:|---:|
| Iterations | 1 | 0 |
| Requests | 25 | 25 |
| Test scripts | 11 | 0 |
| Assertions | 25 | 25 |

Гол алдаа:

```text
connect ECONNREFUSED 127.0.0.1:3000
```

```text
Exit code: 1
Assertions executed: 25
Assertions failed: 25
```

Бүтэн Newman гаралт:

```text
results/newman-down.txt
```

`ECONNREFUSED` нь oracle-ийн хүлээгдэж буй утга буруу гэсэн үг биш. Энэ нь API сервертэй TCP холболт үүсээгүй **interface/connection failure** юм. Харин `newman-fail.txt` дахь алдаа нь серверээс хариу ирсэн боловч зориуд буруу oracle тавьснаас үүссэн **assertion failure** юм.

---

## 9. Репозиторийн бүтэц

```text
lab05/
├── server.js
├── lab05-collection.json
├── lab05-collection-fail.json
├── README.md
├── .gitignore
└── results/
    ├── newman-pass.txt
    ├── newman-fail.txt
    └── newman-down.txt
```

`.gitignore`:

```gitignore
.DS_Store
.idea/
node_modules/
```

---

## 10. Git commit түүх

Лабораторийн ажлыг үе шаттай хийж, 4-өөс олон commit үүсгэсэн.

```text
7717e74 server down newman test hiiv.
5454582 newman aldaatai ajilluulsan.
73e9345 newman api test amjilttai
5cff2a0 postman import hiiv
7a0e3b8 readme zaslav
5e55a7e readme file iin ner soliv.
b050450 readme oruullaa.
0c88e4f Lab05 server amjilttai ajilsan
```

Нийт: **8 commit**

---

## 11. Үр дүнгийн нэгтгэл

| Ажиллуулалт | Requests failed | Assertions failed | Exit code | Тайлбар |
|---|---:|---:|---:|---|
| PASS | 0 | 0 | 0 | Бүх тест амжилттай |
| Intentional FAIL | 0 | 1 | 1 | Oracle зориуд буруу |
| Server DOWN | 25 | 25 | 1 | `ECONNREFUSED`, сервертэй холболтгүй |

Үндсэн PASS ажиллуулалтын баталгаажсан assertion count нь **25** бөгөөд `results/newman-pass.txt` файлтай таарна.

---

## 12. Дүгнэлт

Энэ лабораторийн ажлаар REST API системийг Postman болон Newman ашиглан системтэйгээр тестлэх дадлага хийлээ. Тест дизайны таван алхмаас сонголтуудыг ялгаж, тэдгээрээс төлөөлөх утга болон давхар алдааны хослол гаргах хэсэг хамгийн их бодол шаардсан. Ялангуяа нэг хүсэлтэд хоёр алдаа зэрэг тохиолдоход сервер аль алдааг түрүүлж буцаах нь анхны спецификацид тодорхойгүй байсан. Туршилтаар student болон course хоёулаа байхгүй үед `ERROR_NO_STUDENT` түрүүлж буцдаг болохыг тогтоосон. Мөн inactive student болон хангагдаагүй prerequisite зэрэг тохиолдоход `ERROR_INACTIVE_STUDENT` түрүүлж буцдаг болохыг тогтоосон. Хязгаарын тестүүдээр prerequisite байхгүй course амжилттай бүртгэгдэж, шаардлагатай талбар дутуу үед `ERROR_BAD_REQUEST`, malformed JSON үед `ERROR_BAD_JSON` буцаж байгааг баталгаажуулсан. PASS collection нь 25 assertion-оос 25-ыг амжилттай давсан тул шалгасан нөхцлүүдийн хүрээнд API-ийн үндсэн логик спецификацитай нийцэж байна. Зориудын FAIL туршилт нь буруу oracle Newman-ийн exit code-ийг `1` болгож CI quality gate хэрхэн ажилладгийг харуулсан. Server DOWN туршилтаар `ECONNREFUSED` алдаа гарсан нь assertion failure болон interface failure хоёр өөр төрлийн доголдол болохыг тодорхой харууллаа.
