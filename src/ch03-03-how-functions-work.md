## الدوال (Functions)

تنتشر الدوال في كود Rust. لقد رأيت بالفعل واحدة من أهم الدوال في اللغة: دالة `main` ، وهي نقطة الدخول (entry point) للعديد من البرامج. لقد رأيت أيضاً الكلمة المفتاحية `fn` ، والتي تسمح لك بالتصريح عن دوال جديدة.

يستخدم كود Rust أسلوب _snake case_ كنمط تقليدي لأسماء الدوال والمتغيرات، حيث تكون جميع الحروف صغيرة وتفصل الشرطة السفلية (underscores) بين الكلمات. إليك برنامج يحتوي على مثال لتعريف دالة:

<span class="filename">اسم الملف: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-16-functions/src/main.rs}}
```

نقوم بتعريف دالة في Rust عن طريق إدخال `fn` متبوعة باسم الدالة ومجموعة من الأقواس. تخبر الأقواس المتعرجة (curly brackets) المصرف (compiler) بمكان بداية ونهاية جسم الدالة (function body).

يمكننا استدعاء أي دالة قمنا بتعريفها عن طريق إدخال اسمها متبوعاً بمجموعة من الأقواس. نظراً لأن `another_function` معرفة في البرنامج، يمكن استدعاؤها من داخل دالة `main`. لاحظ أننا قمنا بتعريف `another_function` _بعد_ دالة `main` في الكود المصدري؛ كان بإمكاننا تعريفها قبل ذلك أيضاً. لا يهتم Rust بمكان تعريف دوالك، طالما أنها معرفة في مكان ما في نطاق (scope) يمكن رؤيته من قبل المستدعي (caller).

دعونا نبدأ مشروعاً ثنائياً (binary project) جديداً باسم _functions_ لاستكشاف الدوال بشكل أكبر. ضع مثال `another_function` في ملف _src/main.rs_ وقم بتشغيله. يجب أن ترى المخرجات التالية:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-16-functions/output.txt}}
```

يتم تنفيذ الأسطر بالترتيب الذي تظهر به في دالة `main`. أولاً يتم طباعة رسالة "Hello, world!" ، ثم يتم استدعاء `another_function` وطباعة رسالتها.

### المعلمات (Parameters)

يمكننا تعريف الدوال بحيث تحتوي على _معلمات_ (parameters)، وهي متغيرات خاصة تشكل جزءاً من توقيع الدالة (function’s signature). عندما تحتوي الدالة على parameters، يمكنك تزويدها بقيم ملموسة (concrete values) لتلك المعلمات. تقنياً، تسمى القيم الملموسة _وسائط_ (arguments)، ولكن في المحادثات العادية، يميل الناس إلى استخدام الكلمتين _parameter_ و _argument_ بالتبادل سواء للمتغيرات في تعريف الدالة أو للقيم الملموسة التي يتم تمريرها عند استدعاء الدالة.

في هذا الإصدار من `another_function` نضيف parameter:

<span class="filename">اسم الملف: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/src/main.rs}}
```

جرب تشغيل هذا البرنامج؛ يجب أن تحصل على المخرجات التالية:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/output.txt}}
```

يحتوي التصريح عن `another_function` على parameter واحد باسم `x`. تم تحديد نوع `x` على أنه `i32`. عندما نمرر `5` إلى `another_function` ، يضع ماكرو `println!` القيمة `5` مكان زوج الأقواس المتعرجة الذي يحتوي على `x` في سلسلة التنسيق (format string).

في توقيعات الدوال، _يجب_ عليك التصريح عن نوع كل parameter. هذا قرار متعمد في تصميم Rust: إن اشتراط توضيحات النوع (type annotations) في تعريفات الدوال يعني أن compiler لا يحتاج منك تقريباً لاستخدامها في أي مكان آخر في الكود لمعرفة النوع الذي تقصده. كما يستطيع compiler تقديم رسائل خطأ أكثر فائدة إذا كان يعرف الأنواع التي تتوقعها الدالة.

عند تعريف عدة parameters، افصل بين التصريحات عن المعلمات بفاصلة، هكذا:

<span class="filename">اسم الملف: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/src/main.rs}}
```

ينشئ هذا المثال دالة باسم `print_labeled_measurement` مع اثنين من parameters. المعلمة الأولى تسمى `value` وهي من نوع `i32`. والثانية تسمى `unit_label` وهي من نوع `char`. تقوم الدالة بعد ذلك بطباعة نص يحتوي على كل من `value` و `unit_label`.

دعونا نجرب تشغيل هذا الكود. استبدل البرنامج الموجود حالياً في ملف _src/main.rs_ بمشروع _functions_ الخاص بك بالمثال السابق وقم بتشغيله باستخدام `cargo run`:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/output.txt}}
```

لأننا استدعينا الدالة بالقيمة `5` كقيمة لـ `value` و `'h'` كقيمة لـ `unit_label` ، فإن مخرجات البرنامج تحتوي على تلك القيم.

### الجمل والتعبيرات (Statements and Expressions)

تتكون أجسام الدوال من سلسلة من الجمل (statements) التي تنتهي اختيارياً بتعبير (expression). حتى الآن، لم تتضمن الدوال التي غطيناها تعبيراً ختامياً، لكنك رأيت تعبيراً كجزء من statement. نظراً لأن Rust لغة قائمة على التعبيرات (expression-based language)، فهذا تمييز مهم يجب فهمه. اللغات الأخرى ليس لديها نفس التمييزات، لذا دعونا نلقي نظرة على ماهية statements و expressions وكيف تؤثر اختلافاتهم على أجسام الدوال.

- **الجمل** (Statements) هي تعليمات تؤدي بعض الإجراءات ولا تعيد قيمة.
- **التعبيرات** (Expressions) تؤول إلى قيمة ناتجة.

دعونا نلقي نظرة على بعض الأمثلة.

لقد استخدمنا بالفعل statements و expressions. إن إنشاء متغير وتعيين قيمة له باستخدام الكلمة المفتاحية `let` هو statement. في القائمة 3-1، `let y = 6;` هي statement.

<Listing number="3-1" file-name="src/main.rs" caption="تصريح دالة main يحتوي على جملة واحدة">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-01/src/main.rs}}
```

</Listing>

تعريفات الدوال هي أيضاً statements؛ المثال السابق بأكمله هو statement في حد ذاته. (ومع ذلك، كما سنرى قريباً، فإن استدعاء الدالة ليس statement).

لا تعيد Statements قيماً. لذلك، لا يمكنك تعيين `let` statement لمتغير آخر، كما يحاول الكود التالي القيام به؛ ستحصل على خطأ:

<span class="filename">اسم الملف: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/src/main.rs}}
```

عند تشغيل هذا البرنامج، فإن الخطأ الذي ستحصل عليه سيبدو كالتالي:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/output.txt}}
```

جملة `let y = 6` لا تعيد قيمة، لذا لا يوجد شيء ليرتبط به `x`. هذا يختلف عما يحدث في لغات أخرى، مثل C و Ruby، حيث تعيد عملية التعيين (assignment) قيمة التعيين. في تلك اللغات، يمكنك كتابة `x = y = 6` ويكون لكل من `x` و `y` القيمة `6` ؛ ليس هذا هو الحال في Rust.

تؤول Expressions إلى قيمة وتشكل معظم بقية الكود الذي ستكتبه في Rust. فكر في عملية حسابية، مثل `5 + 6` ، وهي expression يؤول إلى القيمة `11`. يمكن أن تكون Expressions جزءاً من statements: في القائمة 3-1، القيمة `6` في الجملة `let y = 6;` هي expression يؤول إلى القيمة `6`. استدعاء الدالة هو expression. استدعاء الماكرو هو expression. كتلة النطاق (scope block) الجديدة التي يتم إنشاؤها باستخدام الأقواس المتعرجة هي expression، على سبيل المثال:

<span class="filename">اسم الملف: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-20-blocks-are-expressions/src/main.rs}}
```

هذا التعبير:

```rust,ignore
{
    let x = 3;
    x + 1
}
```

هو كتلة (block) تؤول في هذه الحالة إلى `4`. ترتبط هذه القيمة بـ `y` كجزء من `let` statement. لاحظ سطر `x + 1` بدون فاصلة منقوطة (semicolon) في نهايته، وهو ما يختلف عن معظم الأسطر التي رأيتها حتى الآن. لا تتضمن Expressions فواصل منقوطة ختامية. إذا أضفت فاصلة منقوطة إلى نهاية expression، فإنك تحوله إلى statement، وعندها لن يعيد قيمة. ضع ذلك في الاعتبار بينما تستكشف قيم إرجاع الدوال والتعبيرات تالياً.

### دوال ذات قيم إرجاع (Functions with Return Values)

يمكن للدوال إعادة قيم إلى الكود الذي يستدعيها. نحن لا نسمي قيم الإرجاع (return values)، ولكن يجب علينا التصريح عن نوعها بعد سهم (`->`). في Rust، قيمة إرجاع الدالة مرادفة لقيمة التعبير الأخير في كتلة جسم الدالة. يمكنك الإرجاع مبكراً من دالة باستخدام الكلمة المفتاحية `return` وتحديد قيمة، ولكن معظم الدوال تعيد التعبير الأخير بشكل ضمني (implicitly). إليك مثال لدالة تعيد قيمة:

<span class="filename">اسم الملف: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/src/main.rs}}
```

لا توجد استدعاءات دوال، أو ماكرو، أو حتى `let` statements في دالة `five` — فقط الرقم `5` بمفرده. هذه دالة صالحة تماماً في Rust. لاحظ أن نوع إرجاع الدالة محدد أيضاً كـ `-> i32`. جرب تشغيل هذا الكود؛ يجب أن تبدو المخرجات كالتالي:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/output.txt}}
```

الرقم `5` في `five` هو قيمة إرجاع الدالة، ولهذا السبب نوع الإرجاع هو `i32`. دعونا نفحص هذا بمزيد من التفصيل. هناك جزئيتان مهمتان: أولاً، يظهر السطر `let x = five();` أننا نستخدم قيمة إرجاع الدالة لتهيئة متغير. نظراً لأن الدالة `five` تعيد `5` ، فإن هذا السطر هو نفسه السطر التالي:

```rust
let x = 5;
```

ثانياً، دالة `five` ليس لها parameters وتحدد نوع قيمة الإرجاع، لكن جسم الدالة هو `5` وحيد بدون فاصلة منقوطة لأنه expression نريد إعادة قيمته.

دعونا نلقي نظرة على مثال آخر:

<span class="filename">اسم الملف: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-22-function-parameter-and-return/src/main.rs}}
```

سيؤدي تشغيل هذا الكود إلى طباعة `The value of x is: 6`. ولكن ماذا يحدث إذا وضعنا فاصلة منقوطة في نهاية السطر الذي يحتوي على `x + 1` ، محولين إياه من expression إلى statement؟

<span class="filename">اسم الملف: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/src/main.rs}}
```

سيؤدي تصريف هذا الكود إلى حدوث خطأ، كالتالي:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/output.txt}}
```

رسالة الخطأ الرئيسية، `mismatched types` ، تكشف عن المشكلة الجوهرية في هذا الكود. يقول تعريف الدالة `plus_one` أنها ستعيد `i32` ، لكن statements لا تؤول إلى قيمة، وهو ما يتم التعبير عنه بـ `()` ، النوع الوحدوي (unit type). لذلك، لا يتم إرجاع أي شيء، مما يتعارض مع تعريف الدالة ويؤدي إلى حدوث خطأ. في هذه المخرجات، يقدم Rust رسالة للمساعدة في تصحيح هذه المشكلة: يقترح إزالة الفاصلة المنقوطة، مما سيصلح الخطأ.
