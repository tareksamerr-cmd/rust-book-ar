## تعريف الهياكل وإنشاؤها (Defining and Instantiating Structs)

تتشابه الهياكل (structs) مع الصفوف (tuples)، التي نوقشت في قسم ["نوع الصف (Tuple)"][tuples] ، في أن كليهما يحمل عدة قيم مرتبطة. ومثل tuples، يمكن أن تكون أجزاء الـ struct من أنواع مختلفة. وبخلاف tuples، ستقوم في الـ struct بتسمية كل قطعة من البيانات بحيث يكون واضحاً ما تعنيه القيم. إضافة هذه الأسماء تعني أن structs أكثر مرونة من tuples: لست مضطراً للاعتماد على ترتيب البيانات لتحديد قيم مثيل (instance) أو الوصول إليها.

لتعريف struct، ندخل الكلمة المفتاحية `struct` ونسمي الـ struct بالكامل. يجب أن يصف اسم الـ struct أهمية قطع البيانات التي يتم تجميعها معاً. ثم، داخل أقواس متعرجة، نعرف أسماء وأنواع قطع البيانات، والتي نسميها _حقولاً_ (fields). على سبيل المثال، توضح القائمة 5-1 struct يخزن معلومات حول حساب مستخدم.

<Listing number="5-1" file-name="src/main.rs" caption="A `User` struct definition">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-01/src/main.rs:here}}
```

</Listing>

لاستخدام struct بعد تعريفه، ننشئ _مثيلاً_ (instance) من ذلك الـ struct عن طريق تحديد قيم ملموسة لكل من الـ fields. ننشئ instance بذكر اسم الـ struct ثم نضيف أقواس متعرجة تحتوي على أزواج _`key: value`_ ، حيث المفاتيح هي أسماء الـ fields والقيم هي البيانات التي نريد تخزينها في تلك الـ fields. ليس علينا تحديد الـ fields بنفس الترتيب الذي صرحنا به عنها في الـ struct. بمعنى آخر، تعريف الـ struct يشبه قالباً عاماً للنوع، وتقوم الـ instances بملء ذلك القالب ببيانات محددة لإنشاء قيم من ذلك النوع. على سبيل المثال، يمكننا التصريح عن مستخدم معين كما هو موضح في القائمة 5-2.

<Listing number="5-2" file-name="src/main.rs" caption="Creating an instance of the `User` struct">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-02/src/main.rs:here}}
```

</Listing>

للحصول على قيمة محددة من struct، نستخدم ترميز النقطة (dot notation). على سبيل المثال، للوصول إلى عنوان البريد الإلكتروني لهذا المستخدم، نستخدم `user1.email`. إذا كان الـ instance قابلاً للتغيير (mutable)، فيمكننا تغيير قيمة باستخدام dot notation والتعيين في field معين. توضح القائمة 5-3 كيفية تغيير القيمة في حقل `email` لمثيل `User` قابل للتغيير.

<Listing number="5-3" file-name="src/main.rs" caption="Changing the value in the `email` field of a `User` instance">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-03/src/main.rs:here}}
```

</Listing>

لاحظ أن الـ instance بالكامل يجب أن يكون mutable؛ لا تسمح لنا Rust بتمييز حقول معينة فقط كـ mutable. وكما هو الحال مع أي تعبير، يمكننا بناء instance جديد من الـ struct كآخر تعبير في جسم الدالة (function body) لإرجاع ذلك الـ instance الجديد ضمناً.

توضح القائمة 5-4 دالة `build_user` تعيد instance من `User` بالبريد الإلكتروني واسم المستخدم المعطيين. يحصل حقل `active` على القيمة `true` ، ويحصل `sign_in_count` على قيمة `1`.

<Listing number="5-4" file-name="src/main.rs" caption="A `build_user` function that takes an email and username and returns a `User` instance">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-04/src/main.rs:here}}
```

</Listing>

من المنطقي تسمية معاملات الدالة (function parameters) بنفس اسم struct fields ، ولكن الاضطرار إلى تكرار أسماء الحقول والمتغيرات `email` و `username` أمر ممل بعض الشيء. إذا كان للـ struct حقول أكثر، فإن تكرار كل اسم سيصبح أكثر إزعاجاً. لحسن الحظ، هناك اختصار مريح!

<!-- Old headings. Do not remove or links may break. -->

<a id="using-the-field-init-shorthand-when-variables-and-fields-have-the-same-name"></a>

### استخدام اختصار تهيئة الحقول (Using the Field Init Shorthand)

لأن أسماء الـ parameters وأسماء struct fields متطابقة تماماً في القائمة 5-4، يمكننا استخدام صيغة (syntax) _اختصار تهيئة الحقول_ (field init shorthand) لإعادة كتابة `build_user` بحيث تتصرف تماماً بنفس الطريقة ولكن بدون تكرار `username` و `email` ، كما هو موضح في القائمة 5-5.

<Listing number="5-5" file-name="src/main.rs" caption="A `build_user` function that uses field init shorthand because the `username` and `email` parameters have the same name as struct fields">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-05/src/main.rs:here}}
```

</Listing>

هنا، نقوم بإنشاء instance جديد من الـ `User` struct، والذي يحتوي على field باسم `email`. نريد تعيين قيمة حقل `email` إلى القيمة الموجودة في parameter الـ `email` لدالة `build_user`. ولأن حقل `email` و parameter الـ `email` لهما نفس الاسم، نحتاج فقط إلى كتابة `email` بدلاً من `email: email`.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-instances-from-other-instances-with-struct-update-syntax"></a>

### إنشاء مثيلات باستخدام صيغة تحديث الهيكل (Creating Instances with Struct Update Syntax)

غالباً ما يكون من المفيد إنشاء instance جديد من struct يتضمن معظم القيم من instance آخر من نفس النوع، ولكن يغير بعضها. يمكنك القيام بذلك باستخدام صيغة تحديث الهيكل (struct update syntax).

أولاً، في القائمة 5-6 نوضح كيفية إنشاء instance جديد من `User` في `user2` بالطريقة العادية، بدون update syntax. نقوم بتعيين قيمة جديدة لـ `email` ولكن بخلاف ذلك نستخدم نفس القيم من `user1` الذي أنشأناه في القائمة 5-2.

<Listing number="5-6" file-name="src/main.rs" caption="Creating a new `User` instance using all but one of the values from `user1`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-06/src/main.rs:here}}
```

</Listing>

باستخدام struct update syntax، يمكننا تحقيق نفس التأثير بكود أقل، كما هو موضح في القائمة 5-7. تحدد الصيغة `..` أن الـ fields المتبقية التي لم يتم تعيينها صراحة يجب أن يكون لها نفس قيمة الـ fields في الـ instance المعطى.
```
يُنشئ الكود في القائمة 5-7 أيضاً instance في `user2` له قيمة مختلفة لـ `email` ولكن له نفس القيم لحقول `username` و `active` و `sign_in_count` من `user1`. يجب أن تأتي `..user1` في النهاية لتحديد أن أي حقول متبقية يجب أن تحصل على قيمها من الحقول المقابلة في `user1` ، ولكن يمكننا اختيار تحديد قيم لعدد الحقول الذي نريده وبأي ترتيب، بغض النظر عن ترتيب الحقول في تعريف الـ struct.

لاحظ أن struct update syntax يستخدم `=` مثل التعيين؛ هذا لأنه ينقل البيانات، تماماً كما رأينا في قسم ["تفاعل المتغيرات والبيانات مع النقل (Move)"][move]. في هذا المثال، لم يعد بإمكاننا استخدام `user1` بعد إنشاء `user2` لأن الـ `String` في حقل `username` لـ `user1` قد نُقل إلى `user2`. إذا أعطينا `user2` قيم `String` جديدة لكل من `email` و `username` ، وبالتالي استخدمنا فقط قيم `active` و `sign_in_count` من `user1` ، فسيظل `user1` صالحاً بعد إنشاء `user2`. كل من `active` و `sign_in_count` هما أنواع تنفذ سمة (trait) الـ `Copy` ، لذا فإن السلوك الذي ناقشناه في قسم ["بيانات المكدس فقط: النسخ (Copy)"][copy] سينطبق. يمكننا أيضاً الاستمرار في استخدام `user1.email` في هذا المثال، لأن قيمته لم تُنقل خارج `user1`.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-tuple-structs-without-named-fields-to-create-different-types"></a>

### إنشاء أنواع مختلفة باستخدام هياكل الصفوف (Creating Different Types with Tuple Structs)

تدعم Rust أيضاً structs تشبه tuples، وتسمى _هياكل الصفوف_ (tuple structs). تمتلك tuple structs المعنى الإضافي الذي يوفره اسم الـ struct ولكن ليس لها أسماء مرتبطة بحقولها؛ بدلاً من ذلك، لديها فقط أنواع الحقول. تكون tuple structs مفيدة عندما تريد إعطاء الـ tuple بالكامل اسماً وجعل الـ tuple نوعاً مختلفاً عن tuples الأخرى، وعندما يكون تسمية كل حقل كما هو الحال في الـ struct العادي أمراً مطولاً أو زائداً عن الحاجة.

لتعريف tuple struct، ابدأ بالكلمة المفتاحية `struct` واسم الـ struct متبوعاً بالأنواع الموجودة في الـ tuple. على سبيل المثال، هنا نعرف ونستخدم اثنين من tuple structs باسم `Color` و `Point`:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-01-tuple-structs/src/main.rs}}
```

</Listing>

لاحظ أن قيم `black` و `origin` هي أنواع مختلفة لأنها instances من tuple structs مختلفة. كل struct تعرفه هو نوع خاص به، على الرغم من أن الحقول داخل الـ struct قد يكون لها نفس الأنواع. على سبيل المثال، الدالة التي تأخذ parameter من نوع `Color` لا يمكنها أخذ `Point` كـ argument، على الرغم من أن كلا النوعين يتكونان من ثلاث قيم `i32`. بخلاف ذلك، تتشابه instances الـ tuple struct مع tuples في أنه يمكنك تفكيكها (destructure) إلى قطعها الفردية، ويمكنك استخدام `.` متبوعة بالفهرس (index) للوصول إلى قيمة فردية. وبخلاف tuples، تتطلب منك tuple structs تسمية نوع الـ struct عند القيام بـ destructure لها. على سبيل المثال، سنكتب `let Point(x, y, z) = origin;` لتفكيك القيم في نقطة الـ `origin` إلى متغيرات تسمى `x` و `y` و `z`.

<!-- Old headings. Do not remove or links may break. -->

<a id="unit-like-structs-without-any-fields"></a>

### تعريف الهياكل الشبيهة بالوحدة (Defining Unit-Like Structs)

يمكنك أيضاً تعريف structs لا تحتوي على أي حقول! تسمى هذه _الهياكل الشبيهة بالوحدة_ (unit-like structs) لأنها تتصرف بشكل مشابه لـ `()` ، وهو نوع الوحدة (unit type) الذي ذكرناه في قسم ["نوع الصف (Tuple)"][tuples]. يمكن أن تكون unit-like structs مفيدة عندما تحتاج إلى تنفيذ trait على نوع ما ولكن ليس لديك أي بيانات تريد تخزينها في النوع نفسه. سنناقش الـ traits في الفصل العاشر. إليك مثال على التصريح عن struct وحدة باسم `AlwaysEqual` وإنشائه:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-04-unit-like-structs/src/main.rs}}
```

</Listing>

لتعريف `AlwaysEqual` ، نستخدم الكلمة المفتاحية `struct` ، والاسم الذي نريده، ثم فاصلة منقوطة. لا حاجة لأقواس متعرجة أو أقواس عادية! بعد ذلك، يمكننا الحصول على instance من `AlwaysEqual` في متغير `subject` بطريقة مماثلة: باستخدام الاسم الذي عرفناه، بدون أي أقواس متعرجة أو عادية. تخيل أننا سنقوم لاحقاً بتنفيذ سلوك لهذا النوع بحيث يكون كل instance من `AlwaysEqual` مساوياً دائماً لكل instance من أي نوع آخر، ربما للحصول على نتيجة معروفة لأغراض الاختبار. لن نحتاج إلى أي بيانات لتنفيذ هذا السلوك! سترى في الفصل العاشر كيفية تعريف traits وتنفيذها على أي نوع، بما في ذلك unit-like structs.

> ### ملكية بيانات الهيكل (Ownership of Struct Data)
>
> في تعريف `User` struct في القائمة 5-1، استخدمنا نوع الـ `String` المملوك بدلاً من نوع شريحة السلسلة (string slice) `&str`. هذا اختيار متعمد لأننا نريد أن يمتلك كل instance من هذا الـ struct جميع بياناته وأن تكون تلك البيانات صالحة طالما أن الـ struct بالكامل صالح.
>
> من الممكن أيضاً للـ structs تخزين مراجع (references) لبيانات مملوكة لشيء آخر، ولكن القيام بذلك يتطلب استخدام _فترات الحياة_ (lifetimes)، وهي ميزة في Rust سنناقشها في الفصل العاشر. تضمن lifetimes أن البيانات المشار إليها بواسطة struct صالحة طالما أن الـ struct صالح. لنفترض أنك حاولت تخزين reference في struct دون تحديد lifetimes، مثل ما يلي في *src/main.rs*؛ هذا لن يعمل:
>
> <Listing file-name="src/main.rs">
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore,does_not_compile
> struct User {
>     active: bool,
>     username: &str,
>     email: &str,
>     sign_in_count: u64,
> }
>
> fn main() {
>     let user1 = User {
>         active: true,
>         username: "someusername123",
>         email: "someone@example.com",
>         sign_in_count: 1,
>     };
> }
> ```
>
> </Listing>
>
> سيشتكي المترجم (compiler) من أنه يحتاج إلى محددات فترات الحياة (lifetime specifiers):
>
> ```console
> $ cargo run
>    Compiling structs v0.1.0 (file:///projects/structs)
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:3:15
>   |
> 3 |     username: &str,
>   |               ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 ~     username: &'a str,
>   |
>
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:4:12
>   |
> 4 |     email: &str,
>   |            ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 |     username: &str,
> 4 ~     email: &'a str,
>   |
>
> For more information about this error, try `rustc --explain E0106`.
> error: could not compile `structs` (bin "structs") due to 2 previous errors
> ```
>
> في الفصل العاشر، سنناقش كيفية إصلاح هذه الأخطاء بحيث يمكنك تخزين references في structs، ولكن في الوقت الحالي، سنصلح أخطاء مثل هذه باستخدام الأنواع المملوكة مثل `String` بدلاً من references مثل `&str`.

[tuples]: ch03-02-data-types.html#the-tuple-type
[move]: ch04-01-what-is-ownership.html#variables-and-data-interacting-with-move
[copy]: ch04-01-what-is-ownership.html#stack-only-data-copy
```
