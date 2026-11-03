# Лаборатори №8 — Mockito ашиглан хамаарлыг орлуулж тестлэх

**F.CSA313 — Программ хангамжийн чанарын баталгаа ба тест**

- **Оюутны нэр:** О.Оюунжаргал
- **Оюутны код:** В232270019

## Орчин

```
$ java -version
openjdk version "21.0.11" 2026-04-21
OpenJDK Runtime Environment Homebrew (build 21.0.11)
OpenJDK 64-Bit Server VM Homebrew (build 21.0.11, mixed mode, sharing)
```

```
$ mvn -version
Apache Maven 3.9.16 (2bdd9fddda4b155ebf8000e807eb73fd829a51d5)
Maven home: /opt/homebrew/Cellar/maven/3.9.16/libexec
Java version: 21.0.11, vendor: Homebrew, runtime: /usr/local/Cellar/openjdk@21/21.0.11/libexec/openjdk.jdk/Contents/Home
Default locale: en_MN, platform encoding: UTF-8
OS name: "mac os x", version: "26.6", arch: "x86_64", family: "mac"
```

`pom.xml`-д `maven.compiler.release=17` гэж заасан бөгөөд JDK 21 дээр ажиллуулсан. Тест ажиллах үед гарах `byte-buddy-agent` анхааруулга нь Mockito-гийн inline mock maker-ээс үүсдэг хэвийн мэдэгдэл бөгөөд алдаа биш.

## Репозиторийн бүтэц

```
f.csa313-lab8/
├── .gitignore
├── README.md
├── pom.xml
├── src/main/java/mn/edu/must/sqat/
│   ├── Student.java
│   ├── StudentRepository.java
│   ├── EmailSender.java
│   └── CourseRegistrationService.java
├── src/test/java/mn/edu/must/sqat/
│   └── CourseRegistrationServiceTest.java
└── results/
    ├── mvn-test.txt
    ├── mvn-test-mutant-1.txt
    ├── mvn-test-mutant-2.txt
    └── mvn-test-mutant-3.txt
```

## Хэсэг A — Тестүүд

`results/mvn-test.txt` файлын эцсийн үр дүн: **Tests run: 10, Failures: 0, Errors: 0, Skipped: 0 — BUILD SUCCESS**.
Нийт 10 тест нь 7 энгийн тест болон A8 параметржүүлсэн тестийн 3 мөрөөс бүрдэнэ.

| №  | Тестийн нэр | Гол хэрэгсэл |
|----|-------------|--------------|
| A1 | `register_activeStudentWithPrerequisites_returnsOkSavesAndSendsEmail` | `when` + `verify` |
| A2 | `register_unknownStudent_returnsErrorNoStudentAndDoesNotSaveOrEmail` | `verify(..., never())` |
| A3 | `register_inactiveStudent_returnsErrorInactiveAndNeverTouchesEmail` | `verifyNoInteractions` |
| A4 | `register_missingPrerequisite_returnsErrorPrerequisitesAndDoesNotSave` | `when` + `never` |
| A5 | `register_emailSenderThrows_stillReturnsOkAndRegistrationIsSaved` | `doThrow(...).when(...)` |
| A6 | `register_nullArgument_throwsIllegalArgumentAndNoInteractions` | `assertThrows` |
| A7 | `register_success_emailBodyContainsCourseId` | `ArgumentCaptor` |
| A8 | `register_errorCases_returnExpectedCodeAndDoNotSaveOrEmail` (3 мөр) | `@ParameterizedTest` + `@CsvSource` |

### A8 дээр `lenient()` ашигласан шалтгаан

A8-ийн гурван мөрөнд ижил stub тохируулдаг ч мөр бүр кодын өөр замаар явдаг тул «оюутан байхгүй», «оюутан идэвхгүй» мөрүүдэд `courseExists`, `prerequisites` методууд огт дуудагдахгүй. `MockitoExtension`-ий strict stubs горим ашиглагдаагүй stub-ийг `UnnecessaryStubbingException` алдаагаар унагадаг тул эдгээр stub-ийг `lenient()` болгож, тухайн мөрөнд ашиглагдахгүй байхыг зөвшөөрсөн.

## Хэсэг B — Мини мутаци

| Мутант | Өөрчлөлт | Унасан тест | Үр дүн |
|--------|----------|-------------|--------|
| 1 | `if (!student.active()) return "ERROR_INACTIVE_STUDENT";` мөрийг comment болгосон | A3 (`register_inactiveStudent_...`), A8[2] (`active=false` мөр) | Failures: 2, BUILD FAILURE |
| 2 | `repository.saveRegistration(studentId, courseId);` мөрийг comment болгосон | A1 (`register_activeStudentWithPrerequisites_...`), A5 (`register_emailSenderThrows_...`) | Failures: 2, BUILD FAILURE |
| 3 | Эхний `if` нөхцөлөөс `\|\| courseId == null` хэсгийг устгасан | A6 (`register_nullArgument_...`) — A6-г зассаны дараа | Failures: 1, BUILD FAILURE |

Хүснэгтийн мэдээллийг `results/mvn-test-mutant-1..3.txt` файлуудын `[ERROR] Failures:` хэсэгтэй тулгаж шалгасан.

**Мутант 1.** Идэвхгүй оюутны шалгалт алга болсон тул A3 дээр код хичээл байгаа эсэхийг шалгах алхам руу шилжсэн. A3-т `courseExists` stub хийгээгүй тул mock анхдагч утга болох `false`-ийг буцааж, үр дүн нь `ERROR_NO_COURSE` болсон. A8-ийн идэвхгүй оюутны мөрөнд бусад бүх шалгалт давагдаж, `"OK"` буцсан.

**Мутант 2.** A1, A5 хоёр `verify(repository).saveRegistration(...)`-аар хадгалалт хийгдсэнийг шалгадаг тул `Wanted but not invoked` алдаагаар унасан. A7 зөвхөн и-мэйлийн агуулгыг шалгадаг тул энэ мутантыг барихгүй, гэхдээ A1, A5 барьж байгаа тул энэ нь цоорхой биш.

**Мутант 3: тестийн цоорхой ба засвар.** Эхэндээ A6 зөвхөн `register(null, COURSE_ID)` тохиолдлыг шалгадаг байсан бөгөөд `studentId == null` нөхцөл кодонд хэвээр үлдсэн тул мутант хийсний дараа ч бүх тест ногоон гарсан. Энэ нь тестийн цоорхой байсан тул A6-д `assertThrows(IllegalArgumentException.class, () -> service.register(STUDENT_ID, null))` гэсэн хоёр дахь шалгалтыг нэмсэн. Засварын дараа мутант дээр `courseId == null` үед exception шидэгдэхгүй, `repository.findStudent` дуудагдаж `"ERROR_NO_STUDENT"` буцдаг тул A6 унадаг болсон. Энэ гаралтыг `results/mvn-test-mutant-3.txt`-д хадгалсан.

**Хэсэг B-ийн дүгнэлт.** Засварын дараа гурван мутант бүгд дор хаяж нэг тестээр баригдсан. Мутант 3 нь тест хангалттай мэт харагдаж байсан ч `||` нөхцөлийн зөвхөн нэг талыг шалгаж байсныг илрүүлсэн.

## Дүгнэлт

Mock нь тест бүрт `when(...)`-ээр хариуг нь зааж, `verify`-аар ямар дуудлага хийгдсэнийг шалгадаг, Mockito-гийн автоматаар үүсгэдэг орлуулагч объект юм. Харин гараар бичсэн fake нь интерфейсийн хялбаршуулсан бодит хэрэгжүүлэлт бөгөөд жишээ нь `HashMap` дээр суурилсан `InMemoryStudentRepository` байж болно. Fake нь төлөв хадгалж, олон тестэд дахин ашиглагддаг ч өөрөө алдаатай байж болох тул тусдаа засвар үйлчилгээ шаарддаг. Mock нь тухайн тестэд хэрэгтэй зан төлөвийг хурдан тодорхойлж, объектуудын хоорондын харилцааг шалгахад илүү тохиромжтой. `Student` record, `List`, `String` зэрэг энгийн өгөгдлийн объект болон тестлэгдэж буй класс өөрөө mock хийгдэх ёсгүй. Гуравдагч сангийн классыг шууд mock хийхийн оронд өөрийн интерфейсээр бүрхэж, тэр интерфейсийг mock хийх нь зөв. `verify`-г хэт их ашиглавал тест кодын дотоод хэрэгжүүлэлттэй хэт холбогдож, үр дүн нь өөрчлөгдөөгүй байхад refactor хийхэд л унадаг эмзэг тест болно. Тиймээс `verify`-г зөвхөн бүртгэл хадгалагдсан, и-мэйл илгээгдсэн гэх мэт гаднаас харагдах чухал үр дагаврыг шалгахад ашиглах нь зүйтэй.