## كيفية كتابة الاختبارات

**الاختبارات (Tests)** هي دوال Rust (Rust functions) تتحقق من أن الكود غير الاختباري (non-test code) يعمل بالطريقة المتوقعة. عادةً ما تقوم نصوص دوال الاختبار (test functions) بتنفيذ هذه الإجراءات الثلاثة:

- إعداد أي بيانات أو حالة مطلوبة.
- تشغيل الكود الذي تريد اختباره.
- التأكيد (Assert) على أن النتائج هي ما تتوقعه.

دعونا نلقي نظرة على الميزات التي توفرها Rust خصيصًا لكتابة الاختبارات التي تتخذ هذه الإجراءات، والتي تشمل السمة test (test attribute)، وبعض الماكرو (macro)، والسمة should_panic (should_panic attribute).

<!-- Old headings. Do not remove or links may break. -->

<a id="the-anatomy-of-a-test-function"></a>

### هيكلة دوال الاختبار (Structuring Test Functions)

في أبسط صورها، الاختبار في Rust هو دالة مُعلّمة بالسمة `test`. السمات (attribute) هي بيانات وصفية (metadata) حول أجزاء من كود Rust؛ أحد الأمثلة هو السمة `derive` التي استخدمناها مع الهياكل (structs) في الفصل الخامس. لتحويل دالة إلى دالة اختبار، أضف `#[test]` على السطر الذي يسبق `fn`. عند تشغيل الاختبارات الخاصة بك باستخدام أمر `cargo test` (cargo test command)، تقوم Rust ببناء ملف تنفيذي لمشغل الاختبار (test runner binary) يقوم بتشغيل الدوال المُعلّمة ويُبلغ عما إذا كانت كل دالة اختبار تنجح (passes) أو تفشل (fails).

عندما نقوم بإنشاء مشروع مكتبة (library project) جديد باستخدام Cargo، يتم إنشاء وحدة (module) اختبار تحتوي على دالة اختبار تلقائيًا لنا. توفر لك هذه الوحدة قالبًا لكتابة الاختبارات الخاصة بك بحيث لا تضطر إلى البحث عن الهيكلة (structure) والبنية (syntax) الدقيقة في كل مرة تبدأ فيها مشروعًا جديدًا. يمكنك إضافة أي عدد تريده من دوال الاختبار الإضافية وأي عدد تريده من وحدات الاختبار!

سوف نستكشف بعض جوانب كيفية عمل الاختبارات من خلال تجربة قالب الاختبار قبل أن نختبر أي كود فعليًا. بعد ذلك، سنكتب بعض الاختبارات الواقعية التي تستدعي بعض الكود الذي كتبناه وتؤكد أن سلوكه صحيح.

لنقم بإنشاء مشروع مكتبة جديد يسمى `adder` سيقوم بجمع رقمين:

```console
$ cargo new adder --lib
     Created library `adder` project
$ cd adder
```

يجب أن يبدو محتوى ملف *src/lib.rs* في مكتبة `adder` الخاصة بك كما في القائمة 11-1.

<Listing number="11-1" file-name="src/lib.rs" caption="الكود الذي تم إنشاؤه تلقائيًا بواسطة `cargo new`">

<!-- manual-regeneration
cd listings/ch11-writing-automated-tests
rm -rf listing-11-01
cargo new listing-11-01 --lib --name adder
cd listing-11-01
echo "$ cargo test" > output.txt
RUSTFLAGS="-A unused_variables -A dead_code" RUST_TEST_THREADS=1 cargo test >> output.txt 2>&1
git diff output.txt # commit any relevant changes; discard irrelevant ones
cd ../../..
-->

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

</Listing>

يبدأ الملف بدالة `add` مثال حتى يكون لدينا شيء لاختباره.

في الوقت الحالي، دعونا نركز فقط على الدالة `it_works`. لاحظ التعليق التوضيحي `#[test]`: تشير هذه السمة إلى أن هذه دالة اختبار، لذا فإن test runner يعرف أن يتعامل مع هذه الدالة على أنها اختبار. قد يكون لدينا أيضًا دوال non-test في وحدة `tests` للمساعدة في إعداد سيناريوهات شائعة أو تنفيذ عمليات شائعة، لذلك نحتاج دائمًا إلى الإشارة إلى الدوال التي هي tests.

يستخدم نص الدالة المثال الماكرو `assert_eq!` (assert_eq! macro) للتأكيد على أن `result`، الذي يحتوي على نتيجة استدعاء `add` بالرقمين 2 و 2، يساوي 4. يعمل هذا التأكيد كمثال على تنسيق الاختبار النموذجي. لنقم بتشغيله لنرى أن هذا test ينجح.

يقوم أمر `cargo test` بتشغيل جميع tests في مشروعنا، كما هو موضح في القائمة 11-2.

<Listing number="11-2" caption="الخرج من تشغيل الاختبار الذي تم إنشاؤه تلقائيًا">

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-01/output.txt}}
```

</Listing>

قام Cargo بتجميع وتشغيل test. نرى السطر `running 1 test`. يُظهر السطر التالي اسم دالة test التي تم إنشاؤها، والتي تسمى `tests::it_works`، وأن نتيجة تشغيل هذا test هي `ok`. الملخص العام `test result: ok.` يعني أن جميع tests نجحت، والجزء الذي يقرأ `1 passed; 0 failed` يجمع عدد tests التي نجحت أو فشلت.

من الممكن وضع علامة على test كـ "متجاهل" (ignored) بحيث لا يتم تشغيله في حالة معينة؛ سنتناول ذلك في قسم ["تجاهل الاختبارات ما لم يُطلب ذلك تحديدًا"](ignoring) لاحقًا في هذا الفصل. نظرًا لأننا لم نفعل ذلك هنا، يُظهر الملخص `0 ignored`. يمكننا أيضًا تمرير وسيطة إلى أمر `cargo test` لتشغيل tests فقط التي يتطابق اسمها مع سلسلة نصية؛ وهذا ما يسمى التصفية (filtering)، وسنتناوله في قسم ["تشغيل مجموعة فرعية من الاختبارات بالاسم"](subset). هنا، لم نقم بتصفية tests التي يتم تشغيلها، لذا فإن نهاية الملخص تُظهر `0 filtered out`.

الإحصائية `0 measured` مخصصة لاختبارات الأداء (benchmark tests) التي تقيس الأداء. اختبارات الأداء، حتى كتابة هذه السطور، متاحة فقط في Rust الليلية (nightly Rust). راجع [التوثيق حول اختبارات الأداء](bench) لمعرفة المزيد.

الجزء التالي من خرج test الذي يبدأ بـ `Doc-tests adder` مخصص لنتائج أي اختبارات التوثيق (documentation tests). ليس لدينا أي documentation tests بعد، ولكن Rust يمكنها تجميع أي أمثلة كود تظهر في توثيق API (API documentation) الخاص بنا. تساعد هذه الميزة في الحفاظ على تزامن وثائقك وكودك! سنناقش كيفية كتابة documentation tests في قسم ["تعليقات التوثيق كاختبارات"](doc-comments) من الفصل 14. في الوقت الحالي، سنتجاهل خرج `Doc-tests`.

لنبدأ في تخصيص test ليناسب احتياجاتنا الخاصة. أولاً، قم بتغيير اسم الدالة `it_works` إلى اسم مختلف، مثل `exploration`، على النحو التالي:

<span class="filename">اسم الملف: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/src/lib.rs}}
```

ثم، قم بتشغيل `cargo test` مرة أخرى. يُظهر الخرج الآن `exploration` بدلاً من `it_works`:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/output.txt}}
```

الآن سنضيف test آخر، ولكن هذه المرة سنجعل test يفشل! تفشل Tests عندما يحدث `panic` لشيء ما في دالة test. يتم تشغيل كل test في خيط (thread) جديد، وعندما يرى الخيط الرئيسي (main thread) أن خيط test قد مات، يتم وضع علامة على test على أنه FAILED. في الفصل 9، تحدثنا عن أن أبسط طريقة لإحداث `panic` هي استدعاء الماكرو `panic!` (panic! macro). أدخل test الجديد كدالة مسماة `another`، بحيث يبدو ملف *src/lib.rs* الخاص بك كما في القائمة 11-3.

<Listing number="11-3" file-name="src/lib.rs" caption="إضافة test ثانٍ سيفشل لأننا نستدعي الماكرو `panic!`">

```rust,panics,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-03/src/lib.rs}}
```

</Listing>

قم بتشغيل tests مرة أخرى باستخدام `cargo test`. يجب أن يبدو الخرج كما في القائمة 11-4، والتي تُظهر أن test `exploration` الخاص بنا نجح و `another` فشل.

<Listing number="11-4" caption="نتائج الاختبار عندما ينجح test واحد ويفشل test واحد">

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-03/output.txt}}
```

</Listing>

<!-- manual-regeneration
rg panicked listings/ch11-writing-automated-tests/listing-11-03/output.txt
check the line number of the panic matches the line number in the following paragraph
 -->

بدلاً من `ok`، يُظهر السطر `test tests::another` كلمة `FAILED`. يظهر قسمان جديدان بين النتائج الفردية والملخص: يعرض الأول السبب المفصل لفشل كل test. في هذه الحالة، نحصل على التفاصيل التي تفيد بأن `tests::another` فشل لأنه حدث له `panic` مع الرسالة `Make this test fail` في السطر 17 في ملف *src/lib.rs*. يسرد القسم التالي أسماء جميع tests الفاشلة فقط، وهو أمر مفيد عندما يكون هناك الكثير من tests والكثير من خرج test الفاشل المفصل. يمكننا استخدام اسم test فاشل لتشغيل هذا test فقط لتصحيحه بسهولة أكبر؛ سنتحدث أكثر عن طرق تشغيل tests في قسم ["التحكم في كيفية تشغيل الاختبارات"](controlling-how-tests-are-run).

يظهر سطر الملخص في النهاية: بشكل عام، نتيجة test الخاصة بنا هي `FAILED`. كان لدينا test واحد نجح و test واحد فشل.

الآن بعد أن رأيت كيف تبدو نتائج test في سيناريوهات مختلفة، دعنا نلقي نظرة على بعض macros الأخرى بخلاف `panic!` المفيدة في tests.

<!-- Old headings. Do not remove or links may break. -->

<a id="checking-results-with-the-assert-macro"></a>

### التحقق من النتائج باستخدام الماكرو `assert!`

الماكرو `assert!` (assert! macro)، الذي توفره المكتبة القياسية (standard library)، مفيد عندما تريد التأكد من أن بعض الشروط في test يتم تقييمها إلى قيمة منطقية (Boolean) `true`. نعطي الماكرو `assert!` وسيطة يتم تقييمها إلى Boolean. إذا كانت القيمة `true`، فلن يحدث شيء وينجح test. إذا كانت القيمة `false`، يستدعي الماكرو `assert!` الماكرو `panic!` للتسبب في فشل test. يساعدنا استخدام الماكرو `assert!` في التحقق من أن الكود الخاص بنا يعمل بالطريقة التي ننويها.

في الفصل 5، القائمة 5-15، استخدمنا هيكل `Rectangle` ودالة `can_hold`، والتي تتكرر هنا في القائمة 11-5. لنضع هذا الكود في ملف *src/lib.rs*، ثم نكتب بعض tests له باستخدام الماكرو `assert!`.

<Listing number="11-5" file-name="src/lib.rs" caption="هيكل `Rectangle` ودالة `can_hold` الخاصة به من الفصل 5">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-05/src/lib.rs}}
```

</Listing>

تُرجع الدالة `can_hold` قيمة Boolean، مما يعني أنها حالة استخدام مثالية للماكرو `assert!`. في القائمة 11-6، نكتب test يمارس الدالة `can_hold` عن طريق إنشاء مثيل `Rectangle` بعرض 8 وارتفاع 7 والتأكيد على أنه يمكن أن يحتوي على مثيل `Rectangle` آخر بعرض 5 وارتفاع 1.

<Listing number="11-6" file-name="src/lib.rs" caption="اختبار لـ `can_hold` يتحقق مما إذا كان المستطيل الأكبر يمكنه بالفعل احتواء مستطيل أصغر">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-06/src/lib.rs:here}}
```

</Listing>

لاحظ السطر `use super::*;` داخل وحدة `tests`. وحدة `tests` هي وحدة عادية تتبع قواعد الرؤية (visibility rules) المعتادة التي تناولناها في الفصل 7 في قسم ["المسارات للإشارة إلى عنصر في شجرة الوحدة"](paths-for-referring-to-an-item-in-the-module-tree). نظرًا لأن وحدة `tests` هي وحدة داخلية، نحتاج إلى جلب الكود قيد الاختبار في الوحدة الخارجية إلى نطاق (scope) الوحدة الداخلية. نستخدم `glob` هنا، لذا فإن أي شيء نحدده في الوحدة الخارجية متاح لوحدة `tests` هذه.

لقد أطلقنا على test الخاص بنا اسم `larger_can_hold_smaller`، وقمنا بإنشاء مثيلي `Rectangle` اللذين نحتاجهما. بعد ذلك، استدعينا الماكرو `assert!` ومررنا له نتيجة استدعاء `larger.can_hold(&smaller)`. من المفترض أن تُرجع هذه العبارة `true`، لذا يجب أن ينجح test الخاص بنا. دعونا نكتشف ذلك!

```console
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.80s
     Running unittests (target/debug/deps/adder-92938fa227f3322b)

running 1 test
test tests::larger_can_hold_smaller ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 filtered out; finished in 0.00s
```

لقد نجح! لنضف test آخر، هذه المرة نؤكد أن مستطيلاً أصغر لا يمكنه احتواء مستطيل أكبر:

<span class="filename">اسم الملف: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/src/lib.rs:here}}
```

نظرًا لأن النتيجة الصحيحة لدالة `can_hold` في هذه الحالة هي `false`، نحتاج إلى نفي تلك النتيجة قبل تمريرها إلى الماكرو `assert!`. ونتيجة لذلك، سينجح test الخاص بنا إذا أعادت `can_hold` القيمة `false`:

```console
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.80s
     Running unittests (target/debug/deps/adder-92938fa227f3322b)

running 2 tests
test tests::smaller_cannot_hold_larger ... ok
test tests::larger_can_hold_smaller ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 filtered out; finished in 0.00s
```

اثنان tests ينجحان! الآن دعونا نرى ما سيحدث لنتائج test الخاصة بنا عندما نقدم خطأ (bug) في الكود الخاص بنا. سنقوم بتغيير تطبيق دالة `can_hold` عن طريق استبدال علامة أكبر من (`>`) بعلامة أقل من (`<`) عند مقارنة العروض:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/src/lib.rs:here}}
```

تشغيل tests الآن ينتج ما يلي:

```console
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.80s
     Running unittests (target/debug/deps/adder-92938fa227f3322b)

running 2 tests
test tests::smaller_cannot_hold_larger ... ok
test tests::larger_can_hold_smaller ... FAILED

failures:

---- tests::larger_can_hold_smaller stdout ----
thread 'tests::larger_can_hold_smaller' panicked at src/lib.rs:21:5:
assertion failed: larger.can_hold(&smaller)
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace

failures:
    tests::larger_can_hold_smaller

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 filtered out; finished in 0.00s
```

لقد اكتشفت tests الخاصة بنا الـ bug! نظرًا لأن `larger.width` هو `8` و `smaller.width` هو `5`، فإن مقارنة العروض في `can_hold` تُرجع الآن `false`: 8 ليست أقل من 5.

<!-- Old headings. Do not remove or links may break. -->

<a id="testing-equality-with-the-assert_eq-and-assert_ne-macros"></a>

### اختبار المساواة باستخدام `assert_eq!` و `assert_ne!`

تتمثل إحدى الطرق الشائعة للتحقق من الوظائف في اختبار المساواة بين نتيجة الكود قيد الاختبار والقيمة التي تتوقع أن تُرجعها الكود. يمكنك القيام بذلك باستخدام الماكرو `assert!` وتمرير عبارة تستخدم عامل التشغيل `==`. ومع ذلك، يعد هذا test شائعًا جدًا لدرجة أن المكتبة القياسية توفر زوجًا من macros - `assert_eq!` و `assert_ne!` (assert_ne! macro) - لإجراء هذا test بشكل أكثر ملاءمة. تقارن هذه macros وسيطتين للمساواة أو عدم المساواة، على التوالي. ستقوم أيضًا بطباعة القيمتين إذا فشل التأكيد، مما يسهل رؤية *سبب* فشل test؛ على العكس من ذلك، يشير الماكرو `assert!` فقط إلى أنه حصل على قيمة `false` لعبارة `==`، دون طباعة القيم التي أدت إلى القيمة `false`.

في القائمة 11-7، نكتب دالة تسمى `add_two` تضيف `2` إلى معلمتها (parameter)، ثم نختبر هذه الدالة باستخدام الماكرو `assert_eq!`.

<Listing number="11-7" file-name="src/lib.rs" caption="اختبار الدالة `add_two` باستخدام الماكرو `assert_eq!`">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-07/src/lib.rs}}
```

</Listing>

دعونا نتحقق من نجاحه!

```console
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.80s
     Running unittests (target/debug/deps/adder-92938fa227f3322b)

running 1 test
test tests::it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 filtered out; finished in 0.00s
```

نقوم بإنشاء متغير يسمى `result` يحمل نتيجة استدعاء `add_two(2)`. بعد ذلك، نمرر `result` و `4` كوسيطتين للماكرو `assert_eq!`. سطر الخرج لهذا test هو `test tests::it_adds_two ... ok`، ويشير نص `ok` إلى أن test الخاص بنا نجح!

لنقدم bug في الكود الخاص بنا لنرى كيف يبدو `assert_eq!` عندما يفشل. قم بتغيير تطبيق دالة `add_two` لإضافة `3` بدلاً من ذلك:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/src/lib.rs:here}}
```

قم بتشغيل tests مرة أخرى:

```console
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.80s
     Running unittests (target/debug/deps/adder-92938fa227f3322b)

running 1 test
test tests::it_adds_two ... FAILED

failures:

---- tests::it_adds_two stdout ----
thread 'tests::it_adds_two' panicked at src/lib.rs:11:5:
assertion `left == right` failed
  left: `5`
 right: `4`
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace

failures:
    tests::it_adds_two

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 filtered out; finished in 0.00s
```

لقد اكتشف test الخاص بنا الـ bug! فشل test `tests::it_adds_two`، وتخبرنا الرسالة أن التأكيد الذي فشل هو `left == right` وما هي قيمتا `left` و `right`. تساعدنا هذه الرسالة في بدء تصحيح الأخطاء (debugging): كانت الوسيطة `left`، حيث كانت لدينا نتيجة استدعاء `add_two(2)`، هي `5`، لكن الوسيطة `right` كانت `4`. يمكنك أن تتخيل أن هذا سيكون مفيدًا بشكل خاص عندما يكون لدينا الكثير من tests قيد التشغيل.

لاحظ أنه في بعض اللغات وأطر عمل الاختبار (test frameworks)، تسمى المعلمات (parameters) لدوال تأكيد المساواة `expected` و `actual`، ويعد الترتيب الذي نحدد به الوسيطات مهمًا. ومع ذلك، في Rust، تسمى `left` و `right`، ولا يهم الترتيب الذي نحدد به القيمة التي نتوقعها والقيمة التي ينتجها الكود. يمكننا كتابة التأكيد في هذا test على النحو التالي `assert_eq!(4, result)`، مما سيؤدي إلى نفس رسالة الفشل التي تعرض `` assertion `left == right` failed ``.

سينجح الماكرو `assert_ne!` إذا كانت القيمتان اللتان نعطيه إياهما غير متساويتين وسيفشل إذا كانتا متساويتين. يعد هذا الماكرو أكثر فائدة للحالات التي لا نكون فيها متأكدين مما *ستكون* عليه القيمة، ولكننا نعرف ما *يجب ألا* تكون عليه القيمة بالتأكيد. على سبيل المثال، إذا كنا نختبر دالة مضمونة لتغيير مدخلاتها (input) بطريقة ما، ولكن الطريقة التي يتم بها تغيير الـ input تعتمد على يوم الأسبوع الذي نشغل فيه tests الخاصة بنا، فقد يكون أفضل شيء للتأكيد هو أن خرج الدالة لا يساوي الـ input.

تحت السطح، يستخدم الماكرو `assert_eq!` و `assert_ne!` عاملي التشغيل `==` و `!=`، على التوالي. عندما تفشل التأكيدات، تطبع هذه macros وسيطاتها باستخدام تنسيق Debug، مما يعني أن القيم التي تتم مقارنتها يجب أن تطبق السمات (traits) `PartialEq` و `Debug`. تطبق جميع الأنواع البدائية (primitive types) ومعظم أنواع المكتبة القياسية هذه traits. بالنسبة للهياكل (structs) والتعدادات (enums) التي تحددها بنفسك، ستحتاج إلى تطبيق `PartialEq` للتأكيد على مساواة تلك الأنواع. ستحتاج أيضًا إلى تطبيق `Debug` لطباعة القيم عندما يفشل التأكيد. نظرًا لأن كلا الـ traits قابلان للاشتقاق (derivable traits)، كما ذكرنا في القائمة 5-12 في الفصل 5، فعادةً ما يكون الأمر بسيطًا مثل إضافة التعليق التوضيحي `#[derive(PartialEq, Debug)]` إلى تعريف struct أو enum الخاص بك. راجع الملحق ج، ["السمات القابلة للاشتقاق"](derivable-traits)، لمزيد من التفاصيل حول هذه الـ traits القابلة للاشتقاق وغيرها.

### إضافة رسائل فشل مخصصة (Custom Failure Messages)

يمكنك أيضًا إضافة رسالة مخصصة (Custom Failure Messages) ليتم طباعتها مع رسالة الفشل كوسيطات اختيارية للـ macros `assert!` و `assert_eq!` و `assert_ne!`. يتم تمرير أي وسيطات محددة بعد الوسيطات المطلوبة إلى الماكرو `format!` (format! macro) (الذي تمت مناقشته في ["الربط باستخدام `+` أو `format!`"](concatenating) في الفصل 8)، بحيث يمكنك تمرير سلسلة تنسيق (format string) تحتوي على عناصر نائبة (placeholder) `{}` وقيم لوضعها في تلك الـ placeholders. تعد الـ Custom Failure Messages مفيدة لتوثيق ما يعنيه التأكيد؛ عندما يفشل test، سيكون لديك فكرة أفضل عن ماهية المشكلة في الكود.

على سبيل المثال، لنفترض أن لدينا دالة تحيي الأشخاص بالاسم ونريد اختبار أن الاسم الذي نمرره إلى الدالة يظهر في الخرج:

<span class="filename">اسم الملف: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-05-greeter/src/lib.rs}}
```

لم يتم الاتفاق على متطلبات هذا البرنامج بعد، ونحن على يقين من أن نص `Hello` في بداية التحية سيتغير. قررنا أننا لا نريد الاضطرار إلى تحديث test عندما تتغير المتطلبات، لذلك بدلاً من التحقق من المساواة التامة للقيمة التي تم إرجاعها من دالة `greeting`، سنؤكد فقط أن الخرج يحتوي على نص الـ input parameter.

الآن لنقدم bug في هذا الكود عن طريق تغيير `greeting` لاستبعاد `name` لنرى كيف يبدو فشل test الافتراضي:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/src/lib.rs:here}}
```

تشغيل هذا test ينتج ما يلي:

```console
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.80s
     Running unittests (target/debug/deps/adder-92938fa227f3322b)

running 1 test
test tests::greeting_contains_name ... FAILED

failures:

---- tests::greeting_contains_name stdout ----
thread 'tests::greeting_contains_name' panicked at src/lib.rs:13:5:
assertion failed: greeting("Carol").contains("Carol")
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace

failures:
    tests::greeting_contains_name

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 filtered out; finished in 0.00s
```

تشير هذه النتيجة فقط إلى أن التأكيد فشل وعلى أي سطر يوجد التأكيد. ستطبع رسالة فشل أكثر فائدة القيمة من دالة `greeting`. لنضف Custom Failure Message تتكون من format string مع placeholder مملوءة بالقيمة الفعلية التي حصلنا عليها من دالة `greeting`:

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/src/lib.rs:here}}
```

الآن عندما نشغل test، سنحصل على رسالة خطأ أكثر إفادة:

```console
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.80s
     Running unittests (target/debug/deps/adder-92938fa227f3322b)

running 1 test
test tests::greeting_contains_name ... FAILED

failures:

---- tests::greeting_contains_name stdout ----
thread 'tests::greeting_contains_name' panicked at src/lib.rs:13:5:
Greeting did not contain name, value was `Hello!`
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace

failures:
    tests::greeting_contains_name

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 filtered out; finished in 0.00s
```

يمكننا رؤية القيمة التي حصلنا عليها بالفعل في خرج test، مما سيساعدنا في تصحيح الأخطاء (debug) لمعرفة ما حدث بدلاً مما كنا نتوقع حدوثه.

### التحقق من الـ Panics باستخدام `should_panic`

بالإضافة إلى التحقق من قيم الإرجاع (return values)، من المهم التحقق من أن الكود الخاص بنا يتعامل مع حالات الخطأ (error conditions) كما نتوقع. على سبيل المثال، ضع في اعتبارك النوع `Guess` الذي أنشأناه في الفصل 9، القائمة 9-13. يعتمد الكود الآخر الذي يستخدم `Guess` على الضمان بأن مثيلات `Guess` ستحتوي فقط على قيم بين 1 و 100. يمكننا كتابة test يضمن أن محاولة إنشاء مثيل `Guess` بقيمة خارج هذا النطاق يحدث لها `panic`.

نقوم بذلك عن طريق إضافة السمة `should_panic` إلى دالة test الخاصة بنا. ينجح test إذا حدث `panic` للكود داخل الدالة؛ ويفشل test إذا لم يحدث `panic` للكود داخل الدالة.

تُظهر القائمة 11-8 test يتحقق من أن error conditions لـ `Guess::new` تحدث عندما نتوقعها.

<Listing number="11-8" file-name="src/lib.rs" caption="اختبار أن شرطًا ما سيتسبب في `panic!`">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-08/src/lib.rs}}
```

</Listing>

نضع السمة `#[should_panic]` بعد السمة `#[test]` وقبل دالة test التي تنطبق عليها. لننظر إلى النتيجة عندما ينجح هذا test:

```console
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.80s
     Running unittests (target/debug/deps/adder-92938fa227f3322b)

running 1 test
test tests::greater_than_100 ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 filtered out; finished in 0.00s
```

يبدو جيدًا! الآن لنقدم bug في الكود الخاص بنا عن طريق إزالة الشرط الذي سيحدث فيه `panic` للدالة `new` إذا كانت القيمة أكبر من 100:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/src/lib.rs:here}}
```

عندما نشغل test في القائمة 11-8، سيفشل:

```console
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.80s
     Running unittests (target/debug/deps/adder-92938fa227f3322b)

running 1 test
test tests::greater_than_100 ... FAILED

failures:

---- tests::greater_than_100 stdout ----
thread 'tests::greater_than_100' panicked at src/lib.rs:11:5:
test did not panic
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace

failures:
    tests::greater_than_100

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 filtered out; finished in 0.00s
```

لا نحصل على رسالة مفيدة جدًا في هذه الحالة، ولكن عندما ننظر إلى دالة test، نرى أنها مُعلّمة بـ `#[should_panic]`. الفشل الذي حصلنا عليه يعني أن الكود في دالة test لم يتسبب في `panic`.

يمكن أن تكون Tests التي تستخدم `should_panic` غير دقيقة. سينجح test `should_panic` حتى لو حدث `panic` لـ test لسبب مختلف عن السبب الذي كنا نتوقعه. لجعل tests `should_panic` أكثر دقة، يمكننا إضافة معلمة `expected` اختيارية إلى السمة `should_panic`. سيتأكد إطار عمل الاختبار (test harness) من أن رسالة الفشل تحتوي على النص المقدم. على سبيل المثال، ضع في اعتبارك الكود المعدل لـ `Guess` في القائمة 11-9 حيث يحدث `panic` للدالة `new` برسائل مختلفة اعتمادًا على ما إذا كانت القيمة صغيرة جدًا أو كبيرة جدًا.

<Listing number="11-9" file-name="src/lib.rs" caption="اختبار لـ `panic!` مع رسالة panic تحتوي على سلسلة فرعية محددة">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-09/src/lib.rs:here}}
```

</Listing>

سينجح هذا test لأن القيمة التي وضعناها في معلمة `expected` لسمة `should_panic` هي سلسلة فرعية (substring) من الرسالة التي يحدث بها `panic` لدالة `Guess::new`. كان بإمكاننا تحديد رسالة panic بأكملها التي نتوقعها، والتي ستكون في هذه الحالة `Guess value must be less than or equal to 100, got 200`. ما تختاره لتحديده يعتمد على مقدار رسالة panic الفريد أو الديناميكي ومدى دقة test الذي تريده. في هذه الحالة، تكفي سلسلة فرعية من رسالة panic لضمان أن الكود في test...

لنر ما يحدث عندما يفشل test `should_panic` مع رسالة `expected`، لنقدم مرة أخرى bug في الكود الخاص بنا عن طريق تبديل نصوص كتل `if value < 1` و `else if value > 100`:

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/src/lib.rs:here}}
```

هذه المرة عندما نشغل test `should_panic`، سيفشل:

```console
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.80s
     Running unittests (target/debug/deps/adder-92938fa227f3322b)

running 1 test
test tests::greater_than_100 ... FAILED

failures:

---- tests::greater_than_100 stdout ----
thread 'tests::greater_than_100' panicked at src/lib.rs:11:5:
panic message did not contain expected string `less than or equal to 100`
panic message: `Guess value must be greater than or equal to 1, got 200`
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace

failures:
    tests::greater_than_100

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 filtered out; finished in 0.00s
```

تشير رسالة الفشل إلى أن هذا test قد حدث له `panic` بالفعل كما توقعنا، لكن رسالة panic لم تتضمن السلسلة المتوقعة `less than or equal to 100`. كانت رسالة panic التي حصلنا عليها في هذه الحالة هي `Guess value must be greater than or equal to 1, got 200`. الآن يمكننا البدء في معرفة مكان الـ bug الخاص بنا!

### استخدام Result<T, E> في الاختبارات

جميع tests الخاصة بنا حتى الآن يحدث لها `panic` عندما تفشل. يمكننا أيضًا كتابة tests تستخدم النتيجة Result<T, E> (Result<T, E>) ! إليك test من القائمة 11-1، أعيدت كتابته لاستخدام Result<T, E> وإرجاع Err بدلاً من `panic`:

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-10-result-in-tests/src/lib.rs:here}}
```

تتمتع الدالة `it_works` الآن بنوع الإرجاع `Result<(), String>`. في نص الدالة، بدلاً من استدعاء الماكرو `assert_eq!`، نُرجع Ok(()) عندما ينجح test و Err مع `String` بالداخل عندما يفشل test.

تتيح لك كتابة tests بحيث تُرجع Result<T, E> استخدام عامل علامة الاستفهام (question mark operator) في نص tests، والتي يمكن أن تكون طريقة ملائمة لكتابة tests يجب أن تفشل إذا أرجعت أي عملية بداخلها متغير Err.

لا يمكنك استخدام التعليق التوضيحي `#[should_panic]` على tests التي تستخدم Result<T, E>. للتأكيد على أن عملية ما تُرجع متغير Err، *لا* تستخدم question mark operator على قيمة Result<T, E>. بدلاً من ذلك، استخدم `assert!(value.is_err())`.

الآن بعد أن عرفت عدة طرق لكتابة tests، دعنا نلقي نظرة على ما يحدث عندما نشغل tests الخاصة بنا ونستكشف الخيارات المختلفة التي يمكننا استخدامها مع `cargo test`.

[concatenating]: ch08-02-strings.html#concatenating-with--or-format
[bench]: ../unstable-book/library-features/test.html
[ignoring]: ch11-02-running-tests.html#ignoring-tests-unless-specifically-requested
[subset]: ch11-02-running-tests.html#running-a-subset-of-tests-by-name
[controlling-how-tests-are-run]: ch11-02-running-tests.html#controlling-how-tests-are-run
[derivable-traits]: appendix-03-derivable-traits.html
[doc-comments]: ch14-02-publishing-to-crates-io.html#documentation-comments-as-tests
[paths-for-referring-to-an-item-in-the-module-tree]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
