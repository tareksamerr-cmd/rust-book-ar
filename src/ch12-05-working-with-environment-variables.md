## العمل مع متغيرات البيئة (Environment Variables)

سنقوم بتحسين البرنامج الثنائي (Binary) لـ `minigrep` عن طريق إضافة ميزة إضافية: خيار للبحث غير الحساس لحالة الأحرف (Case-insensitive Searching) الذي يمكن للمستخدم تفعيله عبر متغير بيئة (Environment Variable). كان بإمكاننا جعل هذه الميزة خياراً لسطر الأوامر (Command Line Option) ومطالبة المستخدمين بإدخاله في كل مرة يريدون تطبيقه فيها، ولكن بجعله بدلاً من ذلك Environment Variable، فإننا نسمح لمستخدمينا بضبط Environment Variable مرة واحدة وجعل جميع عمليات البحث الخاصة بهم غير حساسة لحالة الأحرف في جلسة الطرفية (Terminal Session) تلك.

<!-- العناوين القديمة. لا تقم بإزالتها وإلا قد تتعطل الروابط. -->
<a id="writing-a-failing-test-for-the-case-insensitive-search-function"></a>

### كتابة اختبار فاشل للبحث غير الحساس لحالة الأحرف

نضيف أولاً دالة (Function) جديدة باسم `search_case_insensitive` إلى مكتبة (Library) `minigrep` والتي سيتم استدعاؤها عندما يكون لـ Environment Variable قيمة. سنستمر في اتباع عملية التطوير القائم على الاختبار (TDD - Test-Driven Development)، لذا فإن الخطوة الأولى هي مرة أخرى كتابة اختبار فاشل (Failing Test). سنضيف اختباراً جديداً لـ Function الجديدة `search_case_insensitive` ونعيد تسمية اختبارنا القديم من `one_result` إلى `case_sensitive` لتوضيح الفروق بين الاختبارين، كما هو موضح في القائمة (Listing) 12-20.

<Listing number="12-20" file-name="src/lib.rs" caption="إضافة اختبار فاشل جديد للدالة غير الحساسة لحالة الأحرف التي أوشكنا على إضافتها">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-20/src/lib.rs:here}}
```

</Listing>

لاحظ أننا قمنا بتحرير محتويات (Contents) الاختبار القديم أيضاً. لقد أضفنا سطراً جديداً بالنص `"Duct tape."` باستخدام حرف _D_ كبير لا ينبغي أن يطابق الاستعلام (Query) `"duct"` عندما نبحث بطريقة حساسة لحالة الأحرف (Case-sensitive). يساعد تغيير الاختبار القديم بهذه الطريقة في ضمان عدم كسر وظيفة البحث الحساس لحالة الأحرف التي قمنا بتنفيذها بالفعل عن طريق الخطأ. يجب أن ينجح هذا الاختبار الآن ويستمر في النجاح بينما نعمل على البحث غير الحساس لحالة الأحرف.

يستخدم الاختبار الجديد للبحث غير الحساس لحالة الأحرف `"rUsT"` كـ Query خاص به. في Function التي أوشكنا على إضافتها `search_case_insensitive` ، يجب أن يطابق Query `"rUsT"` السطر الذي يحتوي على `"Rust:"` بحرف _R_ كبير ويطابق السطر `"Trust me."` على الرغم من أن كليهما لهما حالة أحرف مختلفة عن Query. هذا هو Failing Test الخاص بنا، وسيفشل في التحويل البرمجي (Compile) لأننا لم نقم بعد بتعريف Function `search_case_insensitive`. لا تتردد في إضافة تنفيذ هيكلي (Skeleton Implementation) يعيد دائماً متجهاً (Vector) فارغاً، على غرار الطريقة التي اتبعناها مع Function `search` في Listing 12-16 لرؤية الاختبار وهو Compile ويفشل.

### تنفيذ دالة `search_case_insensitive`

ستكون Function `search_case_insensitive` الموضحة في Listing 12-21، هي نفسها تقريباً Function `search`. الفرق الوحيد هو أننا سنقوم بتحويل Query وكل سطر (Line) إلى أحرف صغيرة (Lowercase) بحيث أياً كانت حالة أحرف وسائط الإدخال (Input Arguments)، فإنها ستكون بنفس حالة الأحرف عندما نتحقق مما إذا كان Line يحتوي على Query.

<Listing number="12-21" file-name="src/lib.rs" caption="تعريف الدالة `search_case_insensitive` لتحويل الاستعلام والسطر إلى أحرف صغيرة قبل مقارنتهما">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-21/src/lib.rs:here}}
```

</Listing>

أولاً، نقوم بتحويل سلسلة (String) الـ Query إلى Lowercase ونخزنها في متغير (Variable) جديد بنفس الاسم، مما يحجب (Shadowing) الـ Query الأصلي. استدعاء `to_lowercase` على Query ضروري بحيث بغض النظر عما إذا كان Query الخاص بالمستخدم هو `"rust"` أو `"RUST"` أو `"Rust"` أو `"rUsT"`، فإننا سنعامل Query كما لو كان `"rust"` ونكون غير حساسين للحالة. بينما سيتعامل `to_lowercase` مع ترميز يونيكود (Unicode) الأساسي، إلا أنه لن يكون دقيقاً بنسبة 100 بالمائة. إذا كنا نكتب تطبيقاً حقيقياً، فسنرغب في القيام بمزيد من العمل هنا، ولكن هذا القسم يتعلق بـ Environment Variables وليس Unicode، لذا سنكتفي بذلك هنا.

لاحظ أن Query هو الآن String بدلاً من شريحة سلسلة (String Slice) لأن استدعاء `to_lowercase` ينشئ بيانات جديدة بدلاً من الإشارة إلى بيانات موجودة. لنفترض أن Query هو `"rUsT"` كمثال: String Slice تلك لا تحتوي على حرف `u` أو `t` صغير لنستخدمه، لذا يتعين علينا تخصيص (Allocate) String جديد يحتوي على `"rust"`. عندما نمرر Query كـ Argument إلى طريقة (Method) `contains` الآن، نحتاج إلى إضافة علامة أند (Ampersand) لأن توقيع (Signature) `contains` مُعرف ليأخذ String Slice.

بعد ذلك، نضيف استدعاءً لـ `to_lowercase` على كل Line لتحويل جميع الأحرف إلى أحرف صغيرة. الآن بعد أن قمنا بتحويل Line و Query إلى Lowercase، سنجد التطابقات بغض النظر عن حالة أحرف Query.

دعونا نرى ما إذا كان هذا التنفيذ يجتاز الاختبارات:

```console
{{#include ../listings/ch12-an-io-project/listing-12-21/output.txt}}
```

رائع! لقد اجتازت الاختبارات. الآن دعونا نستدعي Function الجديدة `search_case_insensitive` من Function `run`. أولاً، سنضيف خيار تكوين (Configuration Option) إلى هيكل (Struct) `Config` للتبديل بين البحث الحساس وغير الحساس لحالة الأحرف. ستؤدي إضافة هذا الحقل (Field) إلى حدوث أخطاء في المترجم (Compiler Errors) لأننا لم نقم بتهيئة (Initializing) هذا Field في أي مكان بعد:

<span class="filename">اسم الملف: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/main.rs:here}}
```

لقد أضفنا Field `ignore_case` الذي يحمل قيمة بولينية (Boolean). بعد ذلك، نحتاج إلى Function `run` للتحقق من قيمة Field `ignore_case` واستخدام ذلك لتقرير ما إذا كان سيتم استدعاء Function `search` أو Function `search_case_insensitive` كما هو موضح في Listing 12-22. هذا لا يزال لن يتم Compile بعد.

<Listing number="12-22" file-name="src/main.rs" caption="استدعاء إما `search` أو `search_case_insensitive` بناءً على القيمة في `config.ignore_case`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/main.rs:there}}
```

</Listing>

أخيراً، نحتاج إلى التحقق من Environment Variable. الدوال المخصصة للعمل مع Environment Variables موجودة في وحدة (Module) `env` في المكتبة القياسية (Standard Library)، والتي هي موجودة بالفعل في النطاق (Scope) في أعلى ملف _src/main.rs_. سنستخدم Function `var` من Module `env` للتحقق مما إذا كان قد تم تعيين أي قيمة لـ Environment Variable يسمى `IGNORE_CASE` كما هو موضح في Listing 12-23.

<Listing number="12-23" file-name="src/main.rs" caption="التحقق من وجود أي قيمة في متغير بيئة يسمى `IGNORE_CASE`">

```rust,ignore,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-23/src/main.rs:here}}
```

</Listing>

هنا، نقوم بإنشاء Variable جديد باسم `ignore_case`. لتعيين قيمته، نستدعي Function `env::var` ونمرر لها اسم Environment Variable `IGNORE_CASE`. تعيد Function `env::var` نتيجة (Result) ستكون متغير (Variant) النجاح `Ok` الذي يحتوي على قيمة Environment Variable إذا تم تعيين Environment Variable لأي قيمة. وستعيد Variant الخطأ `Err` إذا لم يتم تعيين Environment Variable.

نحن نستخدم Method `is_ok` على Result للتحقق مما إذا كان Environment Variable قد تم تعيينه، مما يعني أن البرنامج يجب أن يقوم ببحث غير حساس لحالة الأحرف. إذا لم يتم تعيين Environment Variable `IGNORE_CASE` لأي شيء، فإن `is_ok` ستعيد `false` وسيقوم البرنامج بإجراء بحث حساس لحالة الأحرف. نحن لا نهتم بـ _قيمة_ Environment Variable، فقط ما إذا كان معيناً أو غير معين، لذا نتحقق من `is_ok` بدلاً من استخدام `unwrap` أو `expect` أو أي من الطرق الأخرى التي رأيناها في Result.

نمرر القيمة الموجودة في Variable `ignore_case` إلى مثيل (Instance) `Config` بحيث يمكن لـ Function `run` قراءة تلك القيمة وتقرير ما إذا كان سيتم استدعاء `search_case_insensitive` أو `search` كما نفذنا في Listing 12-22.

دعونا نجرب ذلك! أولاً، سنقوم بتشغيل برنامجنا دون تعيين Environment Variable ومع Query `to` ، والذي يجب أن يطابق أي Line يحتوي على كلمة _to_ بجميع الأحرف الصغيرة:

```console
{{#include ../listings/ch12-an-io-project/listing-12-23/output.txt}}
```

يبدو أن هذا لا يزال يعمل! الآن دعونا نشغل البرنامج مع تعيين `IGNORE_CASE` إلى `1` ولكن بنفس Query `to`:

```console
$ IGNORE_CASE=1 cargo run -- to poem.txt
```

إذا كنت تستخدم PowerShell، فستحتاج إلى تعيين Environment Variable وتشغيل البرنامج كأوامر منفصلة:

```console
PS> $Env:IGNORE_CASE=1; cargo run -- to poem.txt
```

سيؤدي هذا إلى جعل `IGNORE_CASE` يستمر لبقية Terminal Session الخاصة بك. يمكن إلغاء تعيينه باستخدام الأمر (Cmdlet) `Remove-Item`:

```console
PS> Remove-Item Env:IGNORE_CASE
```

يجب أن نحصل على Lines تحتوي على _to_ والتي قد تحتوي على أحرف كبيرة:

<!-- manual-regeneration
cd listings/ch12-an-io-project/listing-12-23
IGNORE_CASE=1 cargo run -- to poem.txt
can't extract because of the environment variable
-->

```console
Are you nobody, too?
How dreary to be somebody!
To tell your name the livelong day
To an admiring bog!
```

ممتاز، لقد حصلنا أيضاً على Lines تحتوي على _To_! يمكن لبرنامج `minigrep` الخاص بنا الآن إجراء بحث غير حساس لحالة الأحرف يتم التحكم فيه بواسطة Environment Variable. الآن أنت تعرف كيفية إدارة الخيارات المحددة باستخدام إما وسائط سطر الأوامر (Command Line Arguments) أو Environment Variables.

تسمح بعض البرامج بـ Arguments و Environment Variables لنفس التكوين. في تلك الحالات، تقرر البرامج أن أحدهما له الأولوية. لتمرين آخر بمفردك، حاول التحكم في حساسية حالة الأحرف من خلال إما Command Line Argument أو Environment Variable. قرر ما إذا كان يجب أن تكون الأولوية لـ Command Line Argument أو Environment Variable إذا تم تشغيل البرنامج مع ضبط أحدهما على الحساسية لحالة الأحرف والآخر على تجاهل حالة الأحرف.

يحتوي Module `std::env` على العديد من الميزات المفيدة الأخرى للتعامل مع Environment Variables: راجع وثائقه (Documentation) لمعرفة ما هو متاح.
