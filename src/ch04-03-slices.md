## نوع الشريحة (The Slice Type)

تتيح لك الـ شرائح (slices) الإشارة إلى تسلسل متصل من العناصر في [مجموعة](ch08-00-common-collections.md)<!-- ignore -->. الشريحة هي نوع من المراجع (references)، لذا فهي لا تمتلك ملكية (ownership).

إليك مشكلة برمجية صغيرة: اكتب دالة تأخذ سلسلة نصية (string) من الكلمات المفصولة بمسافات وتعيد الكلمة الأولى التي تجدها في تلك السلسلة. إذا لم تجد الدالة مسافة في السلسلة، فيجب اعتبار السلسلة بأكملها كلمة واحدة، وبالتالي يجب إعادة السلسلة كاملة.

> ملاحظة: لأغراض تقديم الـ slices، نفترض استخدام ASCII فقط في هذا القسم؛ يوجد نقاش أكثر تفصيلاً حول التعامل مع UTF-8 في قسم ["تخزين النصوص المشفرة بـ UTF-8 باستخدام السلاسل النصية"][strings]<!-- ignore --> في الفصل 8.

دعنا نستعرض كيف سنكتب توقيع (signature) هذه الدالة دون استخدام الـ slices، لفهم المشكلة التي ستحلها الـ slices:

```rust,ignore
fn first_word(s: &String) -> ?
```

تمتلك دالة `first_word` معاملًا (parameter) من نوع `&String`. نحن لا نحتاج إلى الـ ownership، لذا هذا جيد. (في لغة Rust الاصطلاحية، لا تأخذ الدوال ملكية وسائطها (arguments) إلا إذا كانت بحاجة لذلك، وستتضح أسباب ذلك مع تقدمنا). ولكن ماذا يجب أن نعيد؟ ليس لدينا حقًا طريقة للتحدث عن *جزء* من السلسلة النصية. ومع ذلك، يمكننا إعادة فهرس (index) نهاية الكلمة، والذي تشير إليه المسافة. لنحاول ذلك، كما هو موضح في القائمة 4-7.

<Listing number="4-7" file-name="src/main.rs" caption="دالة `first_word` التي تعيد قيمة فهرس بايت داخل معامل الـ `String` ">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:here}}
```

</Listing>

لأننا بحاجة إلى المرور عبر الـ `String` عنصرًا تلو الآخر والتحقق مما إذا كانت القيمة مسافة، سنقوم بتحويل الـ `String` الخاص بنا إلى مصفوفة من البايتات (array of bytes) باستخدام دالة `as_bytes`.

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:as_bytes}}
```

بعد ذلك، ننشئ مكررًا (iterator) فوق مصفوفة البايتات باستخدام دالة `iter`:

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:iter}}
```

سنناقش الـ iterators بمزيد من التفصيل في [الفصل 13][ch13]<!-- ignore -->. في الوقت الحالي، اعلم أن `iter` هي دالة تعيد كل عنصر في المجموعة، وأن `enumerate` تغلف نتيجة `iter` وتعيد كل عنصر كجزء من صف (tuple) بدلاً من ذلك. العنصر الأول في الـ tuple المعاد من `enumerate` هو الـ index، والعنصر الثاني هو مرجع للعنصر. هذا أكثر ملاءمة قليلاً من حساب الـ index بأنفسنا.

لأن دالة `enumerate` تعيد tuple، يمكننا استخدام الأنماط (patterns) لتفكيك ذلك الـ tuple. سنناقش الـ patterns أكثر في [الفصل 6][ch6]<!-- ignore -->. في حلقة `for` نحدد نمطًا يحتوي على `i` للـ index في الـ tuple و `&item` للبايت الواحد في الـ tuple. ولأننا نحصل على مرجع للعنصر من `.iter().enumerate()` نستخدم `&` في النمط.

داخل حلقة `for` نبحث عن البايت الذي يمثل المسافة باستخدام بناء جملة البايت الحرفي (byte literal syntax). إذا وجدنا مسافة، نعيد الموضع. خلاف ذلك، نعيد طول السلسلة باستخدام `s.len()`.

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:inside_for}}
```

لدينا الآن طريقة لمعرفة فهرس نهاية الكلمة الأولى في السلسلة، ولكن هناك مشكلة. نحن نعيد نوع `usize` بمفرده، لكنه لا يكون ذا معنى إلا في سياق الـ `&String`. بعبارة أخرى، لأنه قيمة منفصلة عن الـ `String` فلا يوجد ضمان بأنه سيظل صالحًا في المستقبل. فكر في البرنامج في القائمة 4-8 الذي يستخدم دالة `first_word` من القائمة 4-7.

<Listing number="4-8" file-name="src/main.rs" caption="تخزين النتيجة من استدعاء دالة `first_word` ثم تغيير محتويات الـ `String` ">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-08/src/main.rs:here}}
```

</Listing>

يترجم هذا البرنامج دون أي أخطاء، وسيفعل ذلك أيضًا إذا استخدمنا `word` بعد استدعاء `s.clear()`. ولأن `word` غير متصل بحالة `s` على الإطلاق، فإن `word` لا يزال يحتوي على القيمة `5`. يمكننا استخدام تلك القيمة `5` مع المتغير `s` لمحاولة استخراج الكلمة الأولى، ولكن هذا سيكون خطأً برمجياً (bug) لأن محتويات `s` قد تغيرت منذ أن حفظنا `5` في `word`.

الاضطرار للقلق بشأن خروج الـ index في `word` عن المزامنة مع البيانات في `s` هو أمر ممل وعرضة للخطأ! إدارة هذه الفهارس تكون أكثر هشاشة إذا كتبنا دالة `second_word`. سيضطر توقيعها ليكون هكذا:

```rust,ignore
fn second_word(s: &String) -> (usize, usize) {
```

الآن نحن نتتبع فهرس بداية ونهاية، ولدينا المزيد من القيم التي تم حسابها من بيانات في حالة معينة ولكنها ليست مرتبطة بتلك الحالة على الإطلاق. لدينا ثلاثة متغيرات غير مرتبطة تطفو حولنا وتحتاج إلى الحفاظ على مزامنتها.

لحسن الحظ، لدى Rust حل لهذه المشكلة: شرائح السلاسل النصية (string slices).

### شرائح السلاسل النصية (String Slices)

الـ *string slice* هي مرجع لتسلسل متصل من عناصر الـ `String` وتبدو هكذا:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-17-slice/src/main.rs:here}}
```

بدلاً من مرجع للـ `String` بالكامل، فإن `hello` هو مرجع لجزء من الـ `String` محدد في الجزء الإضافي `[0..5]`. ننشئ الـ slices باستخدام نطاق (range) داخل أقواس مربعة عن طريق تحديد `[starting_index..ending_index]` حيث يكون _`starting_index`_ هو الموضع الأول في الشريحة و _`ending_index`_ هو أكثر بواحد من الموضع الأخير في الشريحة. داخليًا، يخزن هيكل بيانات الشريحة موضع البداية وطول الشريحة، والذي يتوافق مع _`ending_index`_ ناقص _`starting_index`_. لذا، في حالة `let world = &s[6..11];` سيكون `world` عبارة عن شريحة تحتوي على مؤشر (pointer) للبايت عند الفهرس 6 من `s` مع قيمة طول قدرها `5`.

يوضح الشكل 4-7 هذا في رسم تخطيطي.

<img alt="Three tables: a table representing the stack data of s, which points
to the byte at index 0 in a table of the string data &quot;hello world&quot; on
the heap. The third table represents the stack data of the slice world, which
has a length value of 5 and points to byte 6 of the heap data table."
src="img/trpl04-07.svg" class="center" style="width: 50%;" />

<span class="caption">الشكل 4-7: شريحة سلسلة نصية تشير إلى جزء من `String`</span>

مع بناء جملة النطاق `..` في Rust، إذا كنت تريد البدء من الفهرس 0، يمكنك حذف القيمة قبل النقطتين. بعبارة أخرى، هذه متساوية:

```rust
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];
```

وبنفس المنطق، إذا كانت شريحتك تتضمن البايت الأخير من الـ `String` يمكنك حذف الرقم اللاحق. وهذا يعني أن هذه متساوية:

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
```

يمكنك أيضًا حذف كلتا القيمتين لأخذ شريحة من السلسلة بأكملها. لذا، هذه متساوية:

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];
```

> ملاحظة: يجب أن تقع فهارس نطاق الـ string slice عند حدود أحرف UTF-8 صالحة. إذا حاولت إنشاء شريحة سلسلة نصية في منتصف حرف متعدد البايتات، فسيخرج برنامجك بخطأ.

مع وضع كل هذه المعلومات في الاعتبار، دعنا نعيد كتابة `first_word` لتعيد شريحة. النوع الذي يشير إلى "شريحة سلسلة نصية" يكتب كـ `&str`:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-18-first-word-slice/src/main.rs:here}}
```

</Listing>

نحصل على الفهرس لنهاية الكلمة بنفس الطريقة التي فعلناها في القائمة 4-7، من خلال البحث عن أول ظهور للمسافة. عندما نجد مسافة، نعيد شريحة سلسلة نصية باستخدام بداية السلسلة وفهرس المسافة كفهارس البداية والنهاية.

الآن عندما نستدعي `first_word` نحصل على قيمة واحدة مرتبطة بالبيانات الأساسية. تتكون القيمة من مرجع لنقطة بداية الشريحة وعدد العناصر في الشريحة.

إعادة شريحة سيعمل أيضًا مع دالة `second_word`:

```rust,ignore
fn second_word(s: &String) -> &str {
```

لدينا الآن واجهة برمجة تطبيقات (API) مباشرة يصعب العبث بها لأن المترجم سيضمن بقاء المراجع داخل الـ `String` صالحة. تذكر الـ bug في البرنامج في القائمة 4-8، عندما حصلنا على الفهرس لنهاية الكلمة الأولى ثم مسحنا السلسلة فصار الفهرس غير صالح؟ كان ذلك الكود غير صحيح منطقيًا ولكنه لم يظهر أي أخطاء فورية. كانت المشاكل ستظهر لاحقًا إذا واصلنا محاولة استخدام فهرس الكلمة الأولى مع سلسلة فارغة. تجعل الـ slices هذا الـ bug مستحيلاً وتخبرنا في وقت أبكر بكثير أن لدينا مشكلة في كودنا. استخدام نسخة الـ slice من `first_word` سيؤدي إلى خطأ في وقت الترجمة (compile-time error):

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/src/main.rs:here}}
```

</Listing>

إليك خطأ المترجم:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/output.txt}}
```

تذكر من قواعد الاستعارة (borrowing rules) أنه إذا كان لدينا مرجع غير قابل للتغيير (immutable reference) لشيء ما، فلا يمكننا أيضًا أخذ مرجع قابل للتغيير (mutable reference). ولأن `clear` تحتاج إلى تقليص الـ `String` فهي بحاجة للحصول على mutable reference. الـ `println!` بعد استدعاء `clear` تستخدم المرجع في `word` لذا يجب أن يظل الـ immutable reference نشطًا عند تلك النقطة. يمنع Rust وجود الـ mutable reference في `clear` والـ immutable reference في `word` في نفس الوقت، ويفشل الترجمة. لم يجعل Rust الـ API الخاص بنا أسهل في الاستخدام فحسب، بل قضى أيضًا على فئة كاملة من الأخطاء في وقت الترجمة!

<!-- Old headings. Do not remove or links may break. -->

<a id="string-literals-are-slices"></a>

#### السلاسل النصية الحرفية كشرائح (String Literals as Slices)

تذكر أننا تحدثنا عن تخزين السلاسل النصية الحرفية (string literals) داخل الملف الثنائي (binary). الآن بعد أن عرفنا عن الـ slices، يمكننا فهم السلاسل النصية الحرفية بشكل صحيح:

```rust
let s = "Hello, world!";
```

نوع `s` هنا هو `&str`: إنه شريحة تشير إلى تلك النقطة المحددة من الـ binary. وهذا أيضًا هو السبب في أن السلاسل النصية الحرفية غير قابلة للتغيير؛ `&str` هو immutable reference.

#### شرائح السلاسل النصية كمعاملات (String Slices as Parameters)

معرفة أنه يمكنك أخذ شرائح من الحرفيات وقيم الـ `String` يقودنا إلى تحسين آخر على `first_word` وهو توقيعها:

```rust,ignore
fn first_word(s: &String) -> &str {
```

سيكتب مبرمج Rust الأكثر خبرة (Rustacean) التوقيع الموضح في القائمة 4-9 بدلاً من ذلك لأنه يسمح لنا باستخدام نفس الدالة على كل من قيم `&String` وقيم `&str`.

<Listing number="4-9" caption="تحسين دالة `first_word` باستخدام شريحة سلسلة نصية لنوع المعامل `s` ">

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:here}}
```

</Listing>

إذا كان لدينا string slice، يمكننا تمريرها مباشرة. إذا كان لدينا `String` يمكننا تمرير شريحة من الـ `String` أو مرجع للـ `String`. تستفيد هذه المرونة من تحويلات فك المراجع (deref coercions)، وهي ميزة سنغطيها في قسم ["استخدام تحويلات فك المراجع في الدوال والـ methods"][deref-coercions]<!-- ignore --> في الفصل 15.

تعريف دالة لتأخذ string slice بدلاً من مرجع لـ `String` يجعل الـ API الخاص بنا أكثر عمومية وفائدة دون فقدان أي وظائف:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:usage}}
```

</Listing>

### شرائح أخرى (Other Slices)

شرائح السلاسل النصية، كما قد تتخيل، خاصة بالسلاسل النصية. ولكن هناك نوع شريحة أكثر عمومية أيضًا. فكر في هذه المصفوفة:

```rust
let a = [1, 2, 3, 4, 5];
```

تمامًا كما قد نرغب في الإشارة إلى جزء من سلسلة نصية، قد نرغب في الإشارة إلى جزء من مصفوفة. سنفعل ذلك هكذا:

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];

assert_eq!(slice, &[2, 3]);
```

هذه الشريحة لها النوع `&[i32]`. وهي تعمل بنفس الطريقة التي تعمل بها شرائح السلاسل النصية، من خلال تخزين مرجع للعنصر الأول وطول. ستستخدم هذا النوع من الشرائح لجميع أنواع المجموعات الأخرى. سنناقش هذه المجموعات بالتفصيل عندما نتحدث عن المتجهات (vectors) في الفصل 8.

## ملخص (Summary)

تضمن مفاهيم الملكية (ownership)، والاستعارة (borrowing)، والشرائح (slices) سلامة الذاكرة (memory safety) في برامج Rust في وقت الترجمة. تمنحك لغة Rust التحكم في استخدام الذاكرة بنفس الطريقة التي تمنحك إياها لغات برمجة الأنظمة الأخرى. ولكن وجود مالك للبيانات يقوم تلقائيًا بتنظيف تلك البيانات عندما يخرج المالك عن النطاق يعني أنك لست مضطرًا لكتابة وتصحيح كود إضافي للحصول على هذا التحكم.

تؤثر الـ Ownership على كيفية عمل الكثير من الأجزاء الأخرى في Rust، لذا سنتحدث عن هذه المفاهيم بشكل أكبر في بقية الكتاب. دعنا ننتقل إلى الفصل 5 وننظر في تجميع قطع البيانات معًا في هيكل `struct`.

[ch13]: ch13-02-iterators.html
[ch6]: ch06-02-match.html#patterns-that-bind-to-values
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[deref-coercions]: ch15-02-deref.html#using-deref-coercions-in-functions-and-methods
