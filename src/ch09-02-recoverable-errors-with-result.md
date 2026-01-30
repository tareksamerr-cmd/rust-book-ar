## الأخطاء القابلة للاسترداد باستخدام `Result` (Recoverable Errors with Result)

معظم الأخطاء ليست خطيرة بما يكفي لتتطلب إيقاف البرنامج بالكامل. أحياناً عندما تفشل دالة (function)، يكون ذلك لسبب يمكنك تفسيره والاستجابة له بسهولة. على سبيل المثال، إذا حاولت فتح ملف وفشلت تلك العملية لأن الملف غير موجود، فقد ترغب في إنشاء الملف بدلاً من إنهاء العملية (process).

تذكر من قسم ["معالجة الفشل المحتمل باستخدام `Result`"][handle_failure] في الفصل الثاني أن تعداد (enum) الـ `Result` مُعرف بأنه يحتوي على متغيرين (variants)، هما `Ok` و `Err` ، كما يلي:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

الـ `T` والـ `E` هما معاملات أنواع عامة (generic type parameters): سنناقش الأنواع العامة (generics) بمزيد من التفصيل في الفصل العاشر. ما تحتاج لمعرفته الآن هو أن `T` يمثل نوع القيمة التي سيتم إرجاعها في حالة النجاح داخل variant الـ `Ok` ، و `E` يمثل نوع الخطأ الذي سيتم إرجاعه في حالة الفشل داخل variant الـ `Err`. ولأن `Result` يحتوي على معاملات الأنواع العامة هذه، يمكننا استخدام نوع `Result` والدوال المعرفة عليه في العديد من المواقف المختلفة حيث قد تختلف قيمة النجاح وقيمة الخطأ التي نريد إرجاعها.

دعونا نستدعي function تعيد قيمة `Result` لأن الـ function قد تفشل. في القائمة 9-3، نحاول فتح ملف.

<Listing number="9-3" file-name="src/main.rs" caption="Opening a file">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-03/src/main.rs}}
```

</Listing>

نوع الإرجاع لـ `File::open` هو `Result<T, E>`. تم ملء المعامل العام `T` بواسطة تطبيق `File::open` بنوع قيمة النجاح، `std::fs::File` ، وهو مقبض ملف (file handle). نوع `E` المستخدم في قيمة الخطأ هو `std::io::Error`. نوع الإرجاع هذا يعني أن استدعاء `File::open` قد ينجح ويعيد file handle يمكننا القراءة منه أو الكتابة إليه. قد يفشل استدعاء الـ function أيضاً: على سبيل المثال، قد لا يكون الملف موجوداً، أو قد لا نمتلك الإذن للوصول إلى الملف. تحتاج function الـ `File::open` إلى طريقة لإخبارنا ما إذا كانت قد نجحت أو فشلت وفي نفس الوقت تعطينا إما الـ file handle أو معلومات الخطأ. هذه المعلومات هي بالضبط ما ينقله enum الـ `Result`.

في الحالة التي ينجح فيها `File::open` ، ستكون القيمة في المتغير `greeting_file_result` مثيلاً (instance) من `Ok` يحتوي على file handle. وفي الحالة التي يفشل فيها، ستكون القيمة في `greeting_file_result` instance من `Err` يحتوي على مزيد من المعلومات حول نوع الخطأ الذي حدث.

نحتاج إلى الإضافة على الكود في القائمة 9-3 لاتخاذ إجراءات مختلفة اعتماداً على القيمة التي يعيدها `File::open`. توضح القائمة 9-4 إحدى طرق معالجة `Result` باستخدام أداة أساسية، وهي تعبير المطابقة (match expression) الذي ناقشناه في الفصل السادس.

<Listing number="9-4" file-name="src/main.rs" caption="Using a `match` expression to handle the `Result` variants that might be returned">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-04/src/main.rs}}
```

</Listing>

لاحظ أنه، مثل enum الـ `Option` ، تم جلب enum الـ `Result` ومتغيراته إلى النطاق (scope) بواسطة التمهيد (prelude)، لذا لا نحتاج إلى تحديد `Result::` قبل متغيرات `Ok` و `Err` في أذرع (arms) الـ `match`.

عندما تكون النتيجة `Ok` ، سيعيد هذا الكود قيمة `file` الداخلية من variant الـ `Ok` ، ثم نقوم بتعيين قيمة file handle تلك للمتغير `greeting_file`. بعد الـ `match` ، يمكننا استخدام الـ file handle للقراءة أو الكتابة.

الذراع الآخر للـ `match` يعالج الحالة التي نحصل فيها على قيمة `Err` من `File::open`. في هذا المثال، اخترنا استدعاء ماكرو (macro) `panic!`. إذا لم يكن هناك ملف باسم _hello.txt_ في دليلنا الحالي وقمنا بتشغيل هذا الكود، فسنرى المخرجات التالية من macro الـ `panic!`:

```console
{{#include ../listings/ch09-error-handling/listing-09-04/output.txt}}
```

كالعادة، تخبرنا هذه المخرجات بالضبط بما حدث من خطأ.

### المطابقة على أخطاء مختلفة (Matching on Different Errors)

الكود في القائمة 9-4 سيقوم بـ `panic!` بغض النظر عن سبب فشل `File::open`. ومع ذلك، نريد اتخاذ إجراءات مختلفة لأسباب فشل مختلفة. إذا فشل `File::open` لأن الملف غير موجود، نريد إنشاء الملف وإرجاع الـ handle للملف الجديد. إذا فشل `File::open` لأي سبب آخر - على سبيل المثال، لأننا لم نمتلك الإذن لفتح الملف - فلا نزال نريد أن يقوم الكود بـ `panic!` بنفس الطريقة التي فعلها في القائمة 9-4. لهذا، نضيف `match expression` داخلياً، كما هو موضح في القائمة 9-5.

<Listing number="9-5" file-name="src/main.rs" caption="Handling different kinds of errors in different ways">

<!-- ignore this test because otherwise it creates hello.txt which causes other
tests to fail lol -->

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-05/src/main.rs}}
```

</Listing>

نوع القيمة التي يعيدها `File::open` داخل variant الـ `Err` هو `io::Error` ، وهو هيكل (struct) توفره المكتبة القياسية (standard library). هذا الـ struct لديه method تسمى `kind` ، يمكننا استدعاؤها للحصول على قيمة `io::ErrorKind`. يتم توفير enum الـ `io::ErrorKind` بواسطة الـ standard library ويحتوي على variants تمثل الأنواع المختلفة من الأخطاء التي قد تنتج عن عملية `io`. الـ variant الذي نريد استخدامه هو `ErrorKind::NotFound` ، والذي يشير إلى أن الملف الذي نحاول فتحه غير موجود بعد. لذا، نقوم بالمطابقة على `greeting_file_result` ، ولكن لدينا أيضاً مطابقة داخلية على `error.kind()`.

الشرط الذي نريد التحقق منه في الـ match الداخلي هو ما إذا كانت القيمة التي تعيدها `error.kind()` هي variant الـ `NotFound` من enum الـ `ErrorKind`. إذا كانت كذلك، نحاول إنشاء الملف باستخدام `File::create`. ومع ذلك، ولأن `File::create` قد يفشل أيضاً، نحتاج إلى ذراع ثانٍ في الـ match expression الداخلي. عندما لا يمكن إنشاء الملف، يتم طباعة رسالة خطأ مختلفة. يبقى الذراع الثاني للـ match الخارجي كما هو، بحيث يصاب البرنامج بالذعر عند حدوث أي خطأ بخلاف خطأ فقدان الملف.

> #### بدائل لاستخدام `match` مع `Result<T, E>`
>
> هذا الكثير من الـ `match`! تعبير `match` مفيد جداً ولكنه أيضاً بدائي (primitive) للغاية. في الفصل الثالث عشر، ستتعلم عن الإغلاقات (closures)، والتي تُستخدم مع العديد من الـ methods المعرفة على `Result<T, E>`. يمكن أن تكون هذه الـ methods أكثر إيجازاً من استخدام `match` عند التعامل مع قيم `Result<T, E>` في الكود الخاص بك.
>
> على سبيل المثال، إليك طريقة أخرى لكتابة نفس المنطق الموضح في القائمة 9-5، هذه المرة باستخدام closures و method الـ `unwrap_or_else`:
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore
> use std::fs::File;
> use std::io::ErrorKind;
>
> fn main() {
>     let greeting_file = File::open("hello.txt").unwrap_or_else(|error| {
>         if error.kind() == ErrorKind::NotFound {
>             File::create("hello.txt").unwrap_or_else(|error| {
>                 panic!("Problem creating the file: {error:?}");
>             })
>         } else {
>             panic!("Problem opening the file: {error:?}");
>         }
>     });
> }
> ```
>
> على الرغم من أن هذا الكود له نفس سلوك القائمة 9-5، إلا أنه لا يحتوي على أي `match expressions` وهو أكثر نظافة في القراءة. عد إلى هذا المثال بعد قراءة الفصل الثالث عشر وابحث عن method الـ `unwrap_or_else` في توثيق الـ standard library. العديد من هذه الـ methods يمكنها تنظيف `match expressions` الضخمة والمتداخلة عند التعامل مع الأخطاء.

<!-- Old headings. Do not remove or links may break. -->

<a id="shortcuts-for-panic-on-error-unwrap-and-expect"></a>

#### اختصارات للذعر عند حدوث خطأ (Shortcuts for Panic on Error)

استخدام `match` يعمل بشكل جيد بما فيه الكفاية، ولكنه قد يكون مطولاً بعض الشيء ولا يوصل النية (intent) دائماً بشكل جيد. نوع `Result<T, E>` لديه العديد من الدوال المساعدة (helper methods) المعرفة عليه للقيام بمهام متنوعة وأكثر تحديداً. الـ method المسمى `unwrap` هو طريقة اختصار مطبقة تماماً مثل `match expression` الذي كتبناه في القائمة 9-4. إذا كانت قيمة `Result` هي variant الـ `Ok` ، سيعيد `unwrap` القيمة الموجودة داخل `Ok`. وإذا كان `Result` هو variant الـ `Err` ، سيقوم `unwrap` باستدعاء macro الـ `panic!` نيابة عنا. إليك مثال على `unwrap` قيد العمل:

<Listing file-name="src/main.rs">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-04-unwrap/src/main.rs}}
```

</Listing>

إذا قمنا بتشغيل هذا الكود بدون ملف _hello.txt_ ، فسنرى رسالة خطأ من استدعاء `panic!` الذي تقوم به method الـ `unwrap`:

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-04-unwrap
cargo run
copy and paste relevant text
-->

```text
thread 'main' panicked at src/main.rs:4:49:
called `Result::unwrap()` on an `Err` value: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

وبالمثل، تتيح لنا method الـ `expect` اختيار رسالة خطأ الـ `panic!`. استخدام `expect` بدلاً من `unwrap` وتقديم رسائل خطأ جيدة يمكن أن يوصل intent الخاص بك ويجعل تتبع مصدر الـ panic أسهل. صيغة (syntax) الـ `expect` تبدو هكذا:

<Listing file-name="src/main.rs">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-05-expect/src/main.rs}}
```

</Listing>

نستخدم `expect` بنفس طريقة `unwrap`: لإرجاع file handle أو استدعاء macro الـ `panic!`. رسالة الخطأ المستخدمة بواسطة `expect` في استدعائها لـ `panic!` ستكون المعامل (parameter) الذي نمرره لـ `expect` ، بدلاً من رسالة `panic!` الافتراضية التي يستخدمها `unwrap`. إليك كيف يبدو الأمر:

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-05-expect
cargo run
copy and paste relevant text
-->

```text
thread 'main' panicked at src/main.rs:5:10:
hello.txt should be included in this project: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

في الكود ذو جودة الإنتاج، يختار معظم مبرمجي Rust (Rustaceans) استخدام `expect` بدلاً من `unwrap` ويعطون سياقاً أكبر حول سبب توقع نجاح العملية دائماً. بهذه الطريقة، إذا ثبت خطأ افتراضاتك يوماً ما، فستمتلك المزيد من المعلومات لاستخدامها في تصحيح الأخطاء (debugging).

### نشر الأخطاء (Propagating Errors)

عندما يستدعي تطبيق function شيئاً قد يفشل، بدلاً من معالجة الخطأ داخل الـ function نفسها، يمكنك إرجاع الخطأ إلى الكود المستدعي حتى يتمكن من تحديد ما يجب فعله. يُعرف هذا باسم _نشر_ (propagating) الخطأ ويعطي تحكماً أكبر للكود المستدعي، حيث قد تتوفر معلومات أو منطق أكثر يملي كيفية معالجة الخطأ مما هو متاح لديك في سياق الكود الخاص بك.

على سبيل المثال، توضح القائمة 9-6 function تقرأ اسم مستخدم من ملف. إذا لم يكن الملف موجوداً أو لم يمكن قراءته، فستعيد هذه الـ function تلك الأخطاء إلى الكود الذي استدعى الـ function.
```
<Listing number="9-6" file-name="src/main.rs" caption="A function that returns errors to the calling code using `match` ">

```rust
{{#include ../listings/ch09-error-handling/listing-09-06/src/main.rs:here}}
```

</Listing>

يمكن كتابة هذه الـ function بطريقة أقصر بكثير، لكننا سنبدأ بالقيام بالكثير منها يدوياً لاستكشاف معالجة الأخطاء؛ وفي النهاية، سنعرض الطريقة الأقصر. دعونا ننظر إلى نوع الإرجاع للـ function أولاً: `Result<String, io::Error>`. هذا يعني أن الـ function تعيد قيمة من النوع `Result<T, E>` ، حيث تم ملء المعامل العام `T` بالنوع الملموس `String` وتم ملء المعامل العام `E` بالنوع الملموس `io::Error`.

إذا نجحت هذه الـ function دون أي مشاكل، فسيستلم الكود الذي يستدعي هذه الـ function قيمة `Ok` تحمل `String` - وهو `username` الذي قرأته هذه الـ function من الملف. وإذا واجهت هذه الـ function أي مشاكل، فسيستلم الكود المستدعي قيمة `Err` تحمل instance من `io::Error` يحتوي على مزيد من المعلومات حول ماهية المشاكل. لقد اخترنا `io::Error` كنوع إرجاع لهذه الـ function لأن هذا هو نوع قيمة الخطأ التي تعيدها كلتا العمليتين اللتين نستدعيهما في جسم (body) هذه الـ function واللتين قد تفشلان: function الـ `File::open` و method الـ `read_to_string`.

يبدأ body الـ function باستدعاء function الـ `File::open`. ثم نعالج قيمة `Result` باستخدام `match` مشابه للـ `match` في القائمة 9-4. إذا نجح `File::open` ، يصبح file handle الموجود في متغير النمط (pattern variable) المسمى `file` هو القيمة في المتغير القابل للتغيير `username_file` وتستمر الـ function. وفي حالة الـ `Err` ، بدلاً من استدعاء `panic!` ، نستخدم الكلمة المفتاحية `return` للخروج مبكراً من الـ function بالكامل وتمرير قيمة الخطأ من `File::open` ، الموجودة الآن في pattern variable المسمى `e` ، مرة أخرى إلى الكود المستدعي كقيمة خطأ لهذه الـ function.

لذا، إذا كان لدينا file handle في `username_file` ، تقوم الـ function بعد ذلك بإنشاء `String` جديد في المتغير `username` وتستدعي method الـ `read_to_string` على file handle الموجود في `username_file` لقراءة محتويات الملف إلى `username`. يعيد method الـ `read_to_string` أيضاً `Result` لأنه قد يفشل، حتى لو نجح `File::open`. لذا، نحتاج إلى `match` آخر لمعالجة ذلك الـ `Result`: إذا نجح `read_to_string` ، فقد نجحت الـ function الخاصة بنا، ونعيد اسم المستخدم من الملف الموجود الآن في `username` مغلفاً بـ `Ok`. وإذا فشل `read_to_string` ، فإننا نعيد قيمة الخطأ بنفس الطريقة التي أعدنا بها قيمة الخطأ في الـ `match` الذي عالج قيمة إرجاع `File::open`. ومع ذلك، لا نحتاج إلى قول `return` صراحة، لأن هذا هو التعبير الأخير في الـ function.

سيتعامل الكود الذي يستدعي هذا الكود بعد ذلك مع الحصول على إما قيمة `Ok` تحتوي على اسم مستخدم أو قيمة `Err` تحتوي على `io::Error`. الأمر متروك للكود المستدعي ليقرر ما سيفعله بتلك القيم. إذا حصل الكود المستدعي على قيمة `Err` ، فيمكنه استدعاء `panic!` وإيقاف البرنامج، أو استخدام اسم مستخدم افتراضي، أو البحث عن اسم المستخدم من مكان آخر غير الملف، على سبيل المثال. ليس لدينا معلومات كافية عما يحاول الكود المستدعي فعله حقاً، لذا فنحن ننشر جميع معلومات النجاح أو الخطأ للأعلى ليتعامل معها بشكل مناسب.

هذا النمط من نشر الأخطاء شائع جداً في Rust لدرجة أن Rust توفر عامل علامة الاستفهام (question mark operator) `?` لجعل ذلك أسهل.

<!-- Old headings. Do not remove or links may break. -->

<a id="a-shortcut-for-propagating-errors-the--operator"></a>

#### اختصار عامل الـ `?` (The `?` Operator Shortcut)

توضح القائمة 9-7 تطبيقاً لـ `read_username_from_file` له نفس وظيفة القائمة 9-6، ولكن هذا التطبيق يستخدم عامل الـ `?`.

<Listing number="9-7" file-name="src/main.rs" caption="A function that returns errors to the calling code using the `?` operator">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-07/src/main.rs:here}}
```

</Listing>

عامل الـ `?` الموضوع بعد قيمة `Result` مُعرف ليعمل بنفس الطريقة تقريباً مثل `match expressions` التي عرفناها لمعالجة قيم `Result` في القائمة 9-6. إذا كانت قيمة الـ `Result` هي `Ok` ، فسيتم إرجاع القيمة الموجودة داخل الـ `Ok` من هذا التعبير، وسيستمر البرنامج. وإذا كانت القيمة هي `Err` ، فسيتم إرجاع الـ `Err` من الـ function بالكامل كما لو كنا قد استخدمنا الكلمة المفتاحية `return` بحيث يتم نشر قيمة الخطأ إلى الكود المستدعي.

هناك فرق بين ما يفعله `match expression` من القائمة 9-6 وما يفعله عامل الـ `?`: قيم الخطأ التي يتم استدعاء عامل الـ `?` عليها تمر عبر function الـ `from` ، المعرفة في سمة (trait) الـ `From` في الـ standard library، والتي تُستخدم لتحويل القيم من نوع إلى آخر. عندما يستدعي عامل الـ `?` الـ function المسمى `from` ، يتم تحويل نوع الخطأ المستلم إلى نوع الخطأ المعرف في نوع إرجاع الـ function الحالية. هذا مفيد عندما تعيد function نوع خطأ واحداً لتمثيل جميع الطرق التي قد تفشل بها الـ function، حتى لو كانت الأجزاء قد تفشل لأسباب عديدة ومختلفة.

على سبيل المثال، يمكننا تغيير function الـ `read_username_from_file` في القائمة 9-7 لتعيد نوع خطأ مخصصاً باسم `OurError` نقوم بتعريفه. إذا قمنا أيضاً بتعريف `impl From<io::Error> for OurError` لإنشاء instance من `OurError` من `io::Error` ، فإن استدعاءات عامل الـ `?` في body الـ `read_username_from_file` ستستدعي `from` وتحول أنواع الأخطاء دون الحاجة إلى إضافة أي كود آخر إلى الـ function.

في سياق القائمة 9-7، فإن الـ `?` في نهاية استدعاء `File::open` سيعيد القيمة الموجودة داخل `Ok` إلى المتغير `username_file`. وإذا حدث خطأ، فسيقوم عامل الـ `?` بالعودة مبكراً من الـ function بالكامل ويعطي أي قيمة `Err` للكود المستدعي. وينطبق الشيء نفسه على الـ `?` في نهاية استدعاء `read_to_string`.

يزيل عامل الـ `?` الكثير من الكود المتكرر (boilerplate) ويجعل تطبيق هذه الـ function أبسط. يمكننا حتى تقصير هذا الكود أكثر عن طريق ربط (chaining) استدعاءات الـ methods مباشرة بعد الـ `?` ، كما هو موضح في القائمة 9-8.

<Listing number="9-8" file-name="src/main.rs" caption="Chaining method calls after the `?` operator">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-08/src/main.rs:here}}
```

</Listing>

لقد نقلنا إنشاء الـ `String` الجديد في `username` إلى بداية الـ function؛ هذا الجزء لم يتغير. وبدلاً من إنشاء متغير `username_file` ، قمنا بربط استدعاء `read_to_string` مباشرة بنتيجة `File::open("hello.txt")?`. لا يزال لدينا `?` في نهاية استدعاء `read_to_string` ، ولا نزال نعيد قيمة `Ok` تحتوي على `username` عندما ينجح كل من `File::open` و `read_to_string` بدلاً من إرجاع الأخطاء. الوظيفة هي نفسها مرة أخرى كما في القائمة 9-6 والقائمة 9-7؛ هذه مجرد طريقة مختلفة وأكثر راحة (ergonomic) لكتابتها.

توضح القائمة 9-9 طريقة لجعل هذا أقصر باستخدام `fs::read_to_string`.

<Listing number="9-9" file-name="src/main.rs" caption="Using `fs::read_to_string` instead of opening and then reading the file">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-09/src/main.rs:here}}
```

</Listing>

قراءة ملف إلى سلسلة نصية هي عملية شائعة إلى حد ما، لذا توفر الـ standard library الـ function المريح `fs::read_to_string` الذي يفتح الملف، وينشئ `String` جديداً، ويقرأ محتويات الملف، ويضع المحتويات في ذلك الـ `String` ، ويعيده. بالطبع، استخدام `fs::read_to_string` لا يعطينا الفرصة لشرح كل معالجة الأخطاء، لذا قمنا بذلك بالطريقة الأطول أولاً.

<!-- Old headings. Do not remove or links may break. -->

<a id="where-the--operator-can-be-used"></a>

#### أين يمكن استخدام عامل الـ `?` (Where to Use the `?` Operator)

لا يمكن استخدام عامل الـ `?` إلا في الـ functions التي يكون نوع إرجاعها متوافقاً مع القيمة التي يُستخدم عليها الـ `?`. وذلك لأن عامل الـ `?` مُعرف للقيام بعودة مبكرة لقيمة خارج الـ function، بنفس الطريقة التي يعمل بها `match expression` الذي عرفناه في القائمة 9-6. في القائمة 9-6، كان الـ `match` يستخدم قيمة `Result` ، وكان ذراع العودة المبكرة يعيد قيمة `Err(e)`. يجب أن يكون نوع إرجاع الـ function هو `Result` بحيث يكون متوافقاً مع هذا الـ `return`.

في القائمة 9-10، دعونا ننظر إلى الخطأ الذي سنحصل عليه إذا استخدمنا عامل الـ `?` في function الـ `main` بنوع إرجاع غير متوافق مع نوع القيمة التي نستخدم الـ `?` عليها.

<Listing number="9-10" file-name="src/main.rs" caption="Attempting to use the `?` in the `main` function that returns `()` won’t compile.">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-10/src/main.rs}}
```

</Listing>

يفتح هذا الكود ملفاً، وهو ما قد يفشل. يتبع عامل الـ `?` قيمة الـ `Result` التي تعيدها `File::open` ، ولكن function الـ `main` هذه لها نوع إرجاع `()` ، وليس `Result`. عندما نقوم بتجميع هذا الكود، نحصل على رسالة الخطأ التالية:

```console
{{#include ../listings/ch09-error-handling/listing-09-10/output.txt}}
```

يشير هذا الخطأ إلى أنه مسموح لنا فقط باستخدام عامل الـ `?` في function تعيد `Result` أو `Option` أو نوعاً آخر يطبق `FromResidual`.

لإصلاح الخطأ، لديك خياران. أحد الخيارات هو تغيير نوع إرجاع الـ function الخاصة بك ليكون متوافقاً مع القيمة التي تستخدم عامل الـ `?` عليها طالما لم يكن لديك قيود تمنع ذلك. الخيار الآخر هو استخدام `match` أو إحدى methods الـ `Result<T, E>` لمعالجة الـ `Result<T, E>` بأي طريقة مناسبة.

ذكرت رسالة الخطأ أيضاً أنه يمكن استخدام الـ `?` مع قيم `Option<T>` أيضاً. كما هو الحال مع استخدام `?` على `Result` ، يمكنك فقط استخدام `?` على `Option` في function تعيد `Option`. سلوك عامل الـ `?` عند استدعائه على `Option<T>` مشابه لسلوكه عند استدعائه على `Result<T, E>`: إذا كانت القيمة هي `None` ، فسيتم إرجاع الـ `None` مبكراً من الـ function عند تلك النقطة. وإذا كانت القيمة هي `Some` ، فإن القيمة الموجودة داخل الـ `Some` هي القيمة الناتجة عن التعبير، وتستمر الـ function. تحتوي القائمة 9-11 على مثال لـ function تجد الحرف الأخير من السطر الأول في النص المعطى.

<Listing number="9-11" caption="Using the `?` operator on an `Option<T>` value">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-11/src/main.rs:here}}
```

</Listing>

تعيد هذه الـ function القيمة `Option<char>` لأنه من المحتمل وجود حرف هناك، ولكن من المحتمل أيضاً عدم وجوده. يأخذ هذا الكود وسيط (argument) شريحة السلسلة النصية (string slice) المسمى `text` ويستدعي method الـ `lines` عليه، والذي يعيد مكرراً (iterator) على الأسطر في السلسلة النصية. ولأن هذه الـ function تريد فحص السطر الأول، فإنها تستدعي `next` على الـ iterator للحصول على القيمة الأولى منه. إذا كان `text` سلسلة نصية فارغة، فسيؤدي استدعاء `next` هذا إلى إرجاع `None` ، وفي هذه الحالة نستخدم `?` للتوقف وإرجاع `None` من `last_char_of_first_line`. وإذا لم يكن `text` سلسلة نصية فارغة، فسيقوم `next` بإرجاع قيمة `Some` تحتوي على string slice للسطر الأول في `text`.

يقوم الـ `?` باستخراج الـ string slice، ويمكننا استدعاء `chars` على ذلك الـ string slice للحصول على iterator لحروفه. نحن مهتمون بالحرف الأخير في هذا السطر الأول، لذا نستدعي `last` لإرجاع العنصر الأخير في الـ iterator. هذا هو `Option` لأنه من المحتمل أن يكون السطر الأول سلسلة نصية فارغة؛ على سبيل المثال، إذا بدأ `text` بسطر فارغ ولكن لديه حروف في أسطر أخرى، كما في `"\nhi"`. ومع ذلك، إذا كان هناك حرف أخير في السطر الأول، فسيتم إرجاعه في variant الـ `Some`. يعطينا عامل الـ `?` في المنتصف طريقة موجزة للتعبير عن هذا المنطق، مما يسمح لنا بتطبيق الـ function في سطر واحد. إذا لم نتمكن من استخدام عامل الـ `?` على `Option` ، فسنضطر إلى تطبيق هذا المنطق باستخدام المزيد من استدعاءات الـ methods أو `match expression`.

لاحظ أنه يمكنك استخدام عامل الـ `?` على `Result` في function تعيد `Result` ، ويمكنك استخدام عامل الـ `?` على `Option` في function تعيد `Option` ، ولكن لا يمكنك الخلط والمطابقة. لن يقوم عامل الـ `?` تلقائياً بتحويل `Result` إلى `Option` أو العكس؛ في تلك الحالات، يمكنك استخدام methods مثل method الـ `ok` على `Result` أو method الـ `ok_or` على `Option` للقيام بالتحويل صراحة.

حتى الآن، جميع الـ functions المسمى `main` التي استخدمناها تعيد `()`. الـ function المسمى `main` خاص لأنه نقطة الدخول ونقطة الخروج لبرنامج قابل للتنفيذ،
```
وهناك قيود على ما يمكن أن يكون عليه نوع إرجاعها لكي يتصرف البرنامج كما هو متوقع.

لحسن الحظ، يمكن لـ `main` أيضاً إرجاع `Result<(), E>`. تحتوي القائمة 9-12 على الكود من القائمة 9-10، لكننا قمنا بتغيير نوع إرجاع `main` ليكون `Result<(), Box<dyn Error>>` وأضفنا قيمة إرجاع `Ok(())` إلى النهاية. سيتم تجميع هذا الكود الآن.

<Listing number="9-12" file-name="src/main.rs" caption="Changing `main` to return `Result<(), E>` allows the use of the `?` operator on `Result` values.">

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-12/src/main.rs}}
```

</Listing>

النوع `Box<dyn Error>` هو كائن سمة (trait object)، والذي سنتحدث عنه في قسم ["استخدام كائنات السمات للتجريد فوق السلوك المشترك"][trait-objects] في الفصل الثامن عشر. في الوقت الحالي، يمكنك قراءة `Box<dyn Error>` لتعني "أي نوع من الأخطاء". يُسمح باستخدام `?` على قيمة `Result` في function الـ `main` مع نوع الخطأ `Box<dyn Error>` لأنه يسمح بإرجاع أي قيمة `Err` مبكراً. على الرغم من أن body الـ `main` هذا سيعيد فقط أخطاء من النوع `std::io::Error` ، إلا أنه من خلال تحديد `Box<dyn Error>` ، سيبقى هذا التوقيع (signature) صحيحاً حتى لو تمت إضافة المزيد من الكود الذي يعيد أخطاء أخرى إلى body الـ `main`.

عندما تعيد function الـ `main` القيمة `Result<(), E>` ، سيخرج الملف القابل للتنفيذ بقيمة `0` إذا أعادت `main` القيمة `Ok(())` وسيخرج بقيمة غير صفرية إذا أعادت `main` قيمة `Err`. تعيد الملفات القابلة للتنفيذ المكتوبة بلغة C أعداداً صحيحة (integers) عند خروجها: البرامج التي تخرج بنجاح تعيد الـ integer `0` ، والبرامج التي تخطئ تعيد عدداً صحيحاً آخر غير `0`. تعيد Rust أيضاً integers من الملفات القابلة للتنفيذ لتكون متوافقة مع هذا العرف.

قد تعيد function الـ `main` أي أنواع تطبق [سمة `std::process::Termination`][termination] ، والتي تحتوي على function تسمى `report` تعيد `ExitCode`. راجع توثيق الـ standard library لمزيد من المعلومات حول تطبيق سمة `Termination` لأنواعك الخاصة.

الآن بعد أن ناقشنا تفاصيل استدعاء `panic!` أو إرجاع `Result` ، دعونا نعود إلى موضوع كيفية تحديد أيهما مناسب للاستخدام في أي حالات.

[handle_failure]: ch02-00-guessing-game-tutorial.html#handling-potential-failure-with-result
[trait-objects]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
[termination]: ../std/process/trait.Termination.html
```
