<!-- Old headings. Do not remove or links may break. -->

<a id="closures-anonymous-functions-that-can-capture-their-environment"></a>
<a id="closures-anonymous-functions-that-capture-their-environment"></a>

## الإغلاقات (Closures)

الإغلاقات (Closures) في Rust هي دوال مجهولة (anonymous functions) يمكنك حفظها في متغير (variable) أو تمريرها كوسائط (arguments) إلى دوال (functions) أخرى. يمكنك إنشاء الـ Closure في مكان واحد ثم استدعاء الـ Closure في مكان آخر لتقييمها في سياق مختلف. على عكس الـ functions، يمكن للـ Closures التقاط قيم من النطاق (scope) الذي تم تعريفها فيه. سنوضح كيف تسمح ميزات الـ Closure هذه بإعادة استخدام الكود (code reuse) وتخصيص السلوك (behavior customization).

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-an-abstraction-of-behavior-with-closures"></a>
<a id="refactoring-using-functions"></a>
<a id="refactoring-with-closures-to-store-code"></a>
<a id="capturing-the-environment-with-closures"></a>

### التقاط البيئة (Capturing the Environment)

سنقوم أولاً بفحص كيف يمكننا استخدام Closures لالتقاط قيم من البيئة (environment) التي تم تعريفها فيها لاستخدامها لاحقًا. إليك السيناريو: بين الحين والآخر، تقوم شركة القمصان لدينا بإهداء قميص حصري ومحدود الإصدار (exclusive, limited-edition shirt) لشخص ما في قائمة البريد (mailing list) الخاصة بنا كعرض ترويجي. يمكن للأشخاص في الـ mailing list إضافة لونهم المفضل (favorite color) اختياريًا إلى ملفهم الشخصي (profile). إذا كان لدى الشخص الذي تم اختياره للحصول على قميص مجاني لون مفضل محدد، فسيحصل على قميص بهذا اللون. إذا لم يحدد الشخص لونًا مفضلاً، فسيحصل على أي لون تتوفر منه الشركة حاليًا على أكبر عدد.

هناك العديد من الطرق لتنفيذ ذلك. لهذا المثال، سنستخدم enum يسمى `ShirtColor` يحتوي على المتغيرات (variants) `Red` و `Blue` (لتبسيط عدد الألوان المتاحة). نمثل مخزون الشركة (inventory) باستخدام struct يسمى `Inventory` يحتوي على حقل باسم `shirts` يتضمن `Vec<ShirtColor>` يمثل ألوان القمصان المتوفرة حاليًا في المخزون. تقوم الدالة `giveaway` المعرفة على `Inventory` بالحصول على تفضيل لون القميص الاختياري للفائز بالقميص المجاني، وتعيد لون القميص الذي سيحصل عليه الشخص. يظهر هذا الإعداد في القائمة 13-1.

<Listing number="13-1" file-name="src/main.rs" caption="سيناريو إهداء قمصان الشركة">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-01/src/main.rs}}
```

</Listing>

يحتوي الـ `store` المعرف في `main` على قميصين أزرق وقميص أحمر متبقيين لتوزيعهما لهذا العرض الترويجي محدود الإصدار. نستدعي الدالة `giveaway` لمستخدم يفضل قميصًا أحمر ومستخدم ليس لديه أي تفضيل.

مرة أخرى، يمكن تنفيذ هذا الكود بعدة طرق، وهنا، للتركيز على Closures، التزمنا بالمفاهيم التي تعلمتها بالفعل، باستثناء جسم الدالة `giveaway` الذي يستخدم Closure. في الدالة `giveaway`، نحصل على تفضيل المستخدم كـ parameter من نوع `Option<ShirtColor>` ونستدعي الدالة `unwrap_or_else` على `user_preference`. يتم تعريف الدالة [`unwrap_or_else` على `Option<T>`][unwrap-or-else]<!-- ignore --> بواسطة المكتبة القياسية (standard library). تأخذ وسيطًا واحدًا: Closure بدون أي وسائط تعيد قيمة `T` (نفس النوع المخزن في متغير `Some` من الـ `Option<T>`، وفي هذه الحالة `ShirtColor`). إذا كان الـ `Option<T>` هو متغير `Some`، فإن `unwrap_or_else` تعيد القيمة من داخل الـ `Some`. إذا كان الـ `Option<T>` هو متغير `None`، فإن `unwrap_or_else` تستدعي الـ Closure وتعيد القيمة التي أعادتها الـ Closure.

نحدد تعبير الـ Closure `|| self.most_stocked()` كوسيط لـ `unwrap_or_else`. هذه Closure لا تأخذ أي parameters بنفسها (إذا كانت الـ Closure تحتوي على parameters، فستظهر بين علامتي الأنبوب العمودي). جسم الـ Closure يستدعي `self.most_stocked()`. نحن نعرّف الـ Closure هنا، وسيقوم تنفيذ `unwrap_or_else` بتقييم الـ Closure لاحقًا إذا كانت النتيجة مطلوبة.

تشغيل هذا الكود يطبع ما يلي:

```console
{{#include ../listings/ch13-functional-features/listing-13-01/output.txt}}
```

أحد الجوانب المثيرة للاهتمام هنا هو أننا مررنا Closure تستدعي `self.most_stocked()` على مثيل `Inventory` الحالي. لم تكن المكتبة القياسية بحاجة إلى معرفة أي شيء عن أنواع `Inventory` أو `ShirtColor` التي عرفناها، أو المنطق الذي نريد استخدامه في هذا السيناريو. تلتقط الـ Closure مرجعًا غير قابل للتغيير (immutable reference) إلى مثيل `self` `Inventory` وتمرره مع الكود الذي نحدده إلى الدالة `unwrap_or_else`. الـ functions، من ناحية أخرى، غير قادرة على التقاط بيئتها بهذه الطريقة.

<!-- Old headings. Do not remove or links may break. -->

<a id="closure-type-inference-and-annotation"></a>

### استنتاج أنواع Closures وتحديدها (Inferring and Annotating Closure Types)

هناك المزيد من الاختلافات بين الـ functions والـ Closures. لا تتطلب الـ Closures عادةً منك تحديد أنواع الـ parameters أو قيمة الإرجاع (return value) كما تفعل الـ fn functions. تحديد أنواع الـ parameters مطلوب في الـ functions لأن الأنواع هي جزء من واجهة صريحة (explicit interface) مكشوفة لمستخدميك. يعد تحديد هذه الواجهة بصرامة أمرًا مهمًا لضمان اتفاق الجميع على أنواع القيم التي تستخدمها الـ function وتعيدها. الـ Closures، من ناحية أخرى، لا تُستخدم في واجهة مكشوفة كهذه: يتم تخزينها في variables، وتُستخدم دون تسميتها وكشفها لمستخدمي مكتبتنا.

عادةً ما تكون الـ Closures قصيرة وذات صلة فقط ضمن سياق ضيق بدلاً من أي سيناريو عشوائي. ضمن هذه السياقات المحدودة، يمكن للمترجم (compiler) استنتاج أنواع الـ parameters ونوع الإرجاع، على غرار كيفية قدرته على استنتاج أنواع معظم الـ variables (هناك حالات نادرة يحتاج فيها الـ compiler أيضًا إلى تحديد أنواع الـ Closure).

كما هو الحال مع الـ variables، يمكننا إضافة تحديد أنواع (type annotations) إذا أردنا زيادة الوضوح والصراحة على حساب أن تكون أكثر إسهابًا مما هو ضروري تمامًا. سيبدو تحديد الأنواع لـ Closure كما هو موضح في القائمة 13-2. في هذا المثال، نقوم بتعريف Closure وتخزينها في variable بدلاً من تعريف الـ Closure في المكان الذي نمررها فيه كوسيط، كما فعلنا في القائمة 13-1.

<Listing number="13-2" file-name="src/main.rs" caption="إضافة تحديد أنواع اختياري لـ parameter وأنواع قيمة الإرجاع في الـ Closure">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-02/src/main.rs:here}}
```

</Listing>

مع إضافة الـ type annotations، يبدو بناء جملة الـ Closures أكثر تشابهًا مع بناء جملة الـ functions. هنا، نعرّف function تضيف 1 إلى الـ parameter الخاص بها و Closure لها نفس السلوك، للمقارنة. لقد أضفنا بعض المسافات لمواءمة الأجزاء ذات الصلة. يوضح هذا كيف أن بناء جملة الـ Closure يشبه بناء جملة الـ function باستثناء استخدام علامات الأنبوب العمودي وكمية بناء الجملة الاختيارية:

```rust,ignore
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

يُظهر السطر الأول تعريف function ويُظهر السطر الثاني تعريف Closure محدد بالكامل. في السطر الثالث، نزيل الـ type annotations من تعريف الـ Closure. في السطر الرابع، نزيل الأقواس المعقوفة، وهي اختيارية لأن جسم الـ Closure يحتوي على تعبير واحد فقط. كل هذه تعريفات صالحة ستنتج نفس السلوك عند استدعائها. تتطلب الأسطر `add_one_v3` و `add_one_v4` تقييم الـ Closures لتتمكن من الـ compile لأن الأنواع سيتم استنتاجها من استخدامها. هذا مشابه لـ `let v = Vec::new();` التي تحتاج إما إلى type annotations أو قيم من نوع ما ليتم إدراجها في الـ Vec لكي يتمكن Rust من استنتاج النوع.

بالنسبة لتعريفات الـ Closure، سيستنتج الـ compiler نوعًا ملموسًا واحدًا لكل من الـ parameters الخاصة بها ولقيمة الإرجاع الخاصة بها. على سبيل المثال، تُظهر القائمة 13-3 تعريف Closure قصيرة تعيد فقط القيمة التي تتلقاها كـ parameter. هذه الـ Closure ليست مفيدة جدًا باستثناء أغراض هذا المثال. لاحظ أننا لم نضف أي type annotations إلى التعريف. نظرًا لعدم وجود type annotations، يمكننا استدعاء الـ Closure بأي نوع، وهو ما فعلناه هنا باستخدام `String` في المرة الأولى. إذا حاولنا بعد ذلك استدعاء `example_closure` بعدد صحيح (integer)، فسنحصل على خطأ.

<Listing number="13-3" file-name="src/main.rs" caption="محاولة استدعاء Closure تم استنتاج أنواعها بنوعين مختلفين">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-03/src/main.rs:here}}
```

</Listing>

يعطينا الـ compiler هذا الخطأ:

```console
{{#include ../listings/ch13-functional-features/listing-13-03/output.txt}}
```

في المرة الأولى التي نستدعي فيها `example_closure` بقيمة `String`، يستنتج الـ compiler نوع `x` ونوع الإرجاع للـ Closure ليكون `String`. يتم بعد ذلك تثبيت هذه الأنواع في الـ Closure في `example_closure`، ونحصل على خطأ في النوع (type error) عندما نحاول بعد ذلك استخدام نوع مختلف مع نفس الـ Closure.

### التقاط المراجع أو نقل الملكية (Capturing References or Moving Ownership)

يمكن للـ Closures التقاط قيم من بيئتها بثلاث طرق، تتوافق مباشرة مع الطرق الثلاث التي يمكن أن تأخذ بها الـ function parameter: الاقتراض غير القابل للتغيير (borrowing immutably)، الاقتراض القابل للتغيير (borrowing mutably)، وأخذ الملكية (taking ownership). ستقرر الـ Closure أيًا من هذه الطرق ستستخدم بناءً على ما يفعله جسم الـ function بالقيم الملتقطة.

في القائمة 13-4، نعرّف Closure تلتقط مرجعًا غير قابل للتغيير (immutable reference) إلى الـ vector المسمى `list` لأنها تحتاج فقط إلى immutable reference لطباعة القيمة.

<Listing number="13-4" file-name="src/main.rs" caption="تعريف واستدعاء Closure تلتقط immutable reference">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-04/src/main.rs}}
```

</Listing>

يوضح هذا المثال أيضًا أن الـ variable يمكن أن يرتبط بتعريف Closure، ويمكننا لاحقًا استدعاء الـ Closure باستخدام اسم الـ variable والأقواس كما لو كان اسم الـ variable اسم function.

نظرًا لأنه يمكن أن يكون لدينا مراجع متعددة غير قابلة للتغيير (immutable references) لـ `list` في نفس الوقت، يظل `list` متاحًا من الكود قبل تعريف الـ Closure، وبعد تعريف الـ Closure ولكن قبل استدعاء الـ Closure، وبعد استدعاء الـ Closure. هذا الكود يعمل ويطبع:

```console
{{#include ../listings/ch13-functional-features/listing-13-04/output.txt}}
```

بعد ذلك، في القائمة 13-5، نغير جسم الـ Closure بحيث يضيف عنصرًا إلى الـ vector `list`. تلتقط الـ Closure الآن مرجعًا قابلاً للتغيير (mutable reference).

<Listing number="13-5" file-name="src/main.rs" caption="تعريف واستدعاء Closure تلتقط mutable reference">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-05/src/main.rs}}
```

</Listing>

هذا الكود يعمل ويطبع:

```console
{{#include ../listings/ch13-functional-features/listing-13-05/output.txt}}
```

لاحظ أنه لم يعد هناك `println!` بين تعريف واستدعاء الـ Closure `borrows_mutably`: عندما يتم تعريف `borrows_mutably`، فإنه يلتقط mutable reference لـ `list`. لا نستخدم الـ Closure مرة أخرى بعد استدعاء الـ Closure، لذلك ينتهي الاقتراض القابل للتغيير (mutable borrow). بين تعريف الـ Closure واستدعاء الـ Closure، لا يُسمح بالاقتراض غير القابل للتغيير (immutable borrow) للطباعة، لأنه لا يُسمح بأي اقتراض آخر عندما يكون هناك mutable borrow. حاول إضافة `println!` هناك لترى رسالة الخطأ التي تحصل عليها!

إذا كنت تريد أن تجبر الـ Closure على أخذ ملكية (ownership) القيم التي تلتقطها من البيئة (environment) حتى لو لم يكن جسم الـ Closure يحتاج إلى الـ ownership بشكل صارم، يمكنك استخدام الكلمة المفتاحية `move` قبل قائمة الـ parameters.

هذه التقنية مفيدة في الغالب عند تمرير Closure إلى thread جديد لنقل البيانات بحيث يمتلكها الـ thread الجديد. سنناقش الـ threads ولماذا قد ترغب في استخدامها بالتفصيل في الفصل 16 عندما نتحدث عن التزامن (concurrency)، ولكن في الوقت الحالي، دعنا نستكشف بإيجاز إنشاء thread جديد باستخدام Closure تحتاج إلى الكلمة المفتاحية `move`. تُظهر القائمة 13-6 القائمة 13-4 معدلة لطباعة الـ vector في thread جديد بدلاً من الـ main thread.

<Listing number="13-6" file-name="src/main.rs" caption="استخدام `move` لإجبار الـ Closure الخاصة بالـ thread على أخذ ownership لـ `list`">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-06/src/main.rs}}
```

</Listing>

نقوم بإنشاء thread جديد، ونعطي الـ thread Closure لتشغيلها كوسيط. يطبع جسم الـ Closure الـ list. في القائمة 13-4، التقطت الـ Closure الـ `list` فقط باستخدام immutable reference لأن هذا هو أقل قدر من الوصول إلى `list` المطلوب لطباعته. في هذا المثال، على الرغم من أن جسم الـ Closure لا يزال يحتاج فقط إلى immutable reference، نحتاج إلى تحديد أنه يجب نقل `list` إلى الـ Closure عن طريق وضع الكلمة المفتاحية `move` في بداية تعريف الـ Closure. إذا أجرى الـ main thread المزيد من العمليات قبل استدعاء `join` على الـ thread الجديد، فقد ينتهي الـ thread الجديد قبل أن ينتهي باقي الـ main thread، أو قد ينتهي الـ main thread أولاً. إذا احتفظ الـ main thread بـ ownership لـ `list` ولكنه انتهى قبل الـ thread الجديد وأسقط `list`، فسيكون الـ immutable reference في الـ thread غير صالح. لذلك، يتطلب الـ compiler نقل `list` إلى الـ Closure المعطاة للـ thread الجديد بحيث يكون الـ reference صالحًا. حاول إزالة الكلمة المفتاحية `move` أو استخدام `list` في الـ main thread بعد تعريف الـ Closure لترى أخطاء الـ compiler التي تحصل عليها!

<!-- Old headings. Do not remove or links may break. -->

<a id="storing-closures-using-generic-parameters-and-the-fn-traits"></a>
<a id="limitations-of-the-cacher-implementation"></a>
<a id="moving-captured-values-out-of-the-closure-and-the-fn-traits"></a>
<a id="moving-captured-values-out-of-closures-and-the-fn-traits"></a>

### نقل القيم الملتقطة خارج Closures (Moving Captured Values Out of Closures)

بمجرد أن تلتقط الـ Closure مرجعًا أو تلتقط ownership لقيمة من البيئة التي تم تعريف الـ Closure فيها (وبالتالي تؤثر على ما يتم نقله _إلى_ الـ Closure، إن وجد)، فإن الكود الموجود في جسم الـ Closure يحدد ما يحدث للمراجع أو القيم عند تقييم الـ Closure لاحقًا (وبالتالي يؤثر على ما يتم نقله _خارج_ الـ Closure، إن وجد).

يمكن لجسم الـ Closure القيام بأي مما يلي: نقل قيمة ملتقطة خارج الـ Closure، تغيير القيمة الملتقطة (mutate the captured value)، لا نقل ولا تغيير للقيمة، أو عدم التقاط أي شيء من البيئة في البداية.

تؤثر الطريقة التي تلتقط بها الـ Closure القيم من البيئة وتتعامل معها على الـ traits التي تنفذها الـ Closure، والـ traits هي كيف يمكن للـ functions والـ structs تحديد أنواع الـ Closures التي يمكنها استخدامها. ستقوم الـ Closures تلقائيًا بتنفيذ واحد أو اثنين أو كل ثلاثة من الـ Fn traits هذه، بطريقة إضافية، اعتمادًا على كيفية تعامل جسم الـ Closure مع القيم:

*   `FnOnce` تنطبق على الـ Closures التي يمكن استدعاؤها مرة واحدة. تنفذ جميع الـ Closures هذا الـ trait على الأقل لأنه يمكن استدعاء جميع الـ Closures. الـ Closure التي تنقل القيم الملتقطة خارج جسمها ستنفذ `FnOnce` فقط وليس أيًا من الـ Fn traits الأخرى لأنه لا يمكن استدعاؤها إلا مرة واحدة.
*   `FnMut` تنطبق على الـ Closures التي لا تنقل القيم الملتقطة خارج جسمها ولكن قد تغير القيم الملتقطة. يمكن استدعاء هذه الـ Closures أكثر من مرة.
*   `Fn` تنطبق على الـ Closures التي لا تنقل القيم الملتقطة خارج جسمها ولا تغير القيم الملتقطة، وكذلك الـ Closures التي لا تلتقط أي شيء من بيئتها. يمكن استدعاء هذه الـ Closures أكثر من مرة دون تغيير بيئتها، وهو أمر مهم في حالات مثل استدعاء Closure عدة مرات بالتزامن (concurrently).

دعنا نلقي نظرة على تعريف الدالة `unwrap_or_else` على `Option<T>` التي استخدمناها في القائمة 13-1:

```rust,ignore
impl<T> Option<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}
```

تذكر أن `T` هو النوع العام (generic type) الذي يمثل نوع القيمة في متغير `Some` من الـ `Option`. هذا النوع `T` هو أيضًا نوع الإرجاع لـ function `unwrap_or_else`: الكود الذي يستدعي `unwrap_or_else` على `Option<String>`، على سبيل المثال، سيحصل على `String`.

بعد ذلك، لاحظ أن function `unwrap_or_else` لديها الـ generic type parameter الإضافي `F`. النوع `F` هو نوع الـ parameter المسمى `f`، وهو الـ Closure الذي نوفره عند استدعاء `unwrap_or_else`.

الـ trait bound المحدد على الـ generic type `F` هو `FnOnce() -> T`، مما يعني أن `F` يجب أن تكون قادرة على الاستدعاء مرة واحدة، ولا تأخذ أي وسائط، وتعيد `T`. استخدام `FnOnce` في الـ trait bound يعبر عن القيد بأن `unwrap_or_else` لن تستدعي `f` أكثر من مرة. في جسم `unwrap_or_else`، يمكننا أن نرى أنه إذا كان الـ `Option` هو `Some`، فلن يتم استدعاء `f`. إذا كان الـ `Option` هو `None`، فسيتم استدعاء `f` مرة واحدة. نظرًا لأن جميع الـ Closures تنفذ `FnOnce`، فإن `unwrap_or_else` تقبل جميع الأنواع الثلاثة من الـ Closures وتكون مرنة قدر الإمكان.

> ملاحظة: إذا كان ما نريد القيام به لا يتطلب التقاط قيمة من البيئة، يمكننا استخدام اسم function بدلاً من Closure حيث نحتاج إلى شيء ينفذ أحد الـ Fn traits. على سبيل المثال، على قيمة `Option<Vec<T>>`، يمكننا استدعاء `unwrap_or_else(Vec::new)` للحصول على vector جديد وفارغ إذا كانت القيمة `None`. يقوم الـ compiler تلقائيًا بتنفيذ أي من الـ Fn traits القابلة للتطبيق لتعريف function.

الآن دعنا نلقي نظرة على دالة المكتبة القياسية `sort_by_key`، المعرفة على slices، لنرى كيف تختلف عن `unwrap_or_else` ولماذا تستخدم `sort_by_key` الـ `FnMut` بدلاً من `FnOnce` لـ trait bound. تحصل الـ Closure على وسيط واحد في شكل مرجع (reference) إلى العنصر الحالي في الـ slice الذي يتم النظر فيه، وتعيد قيمة من نوع `K` يمكن ترتيبها. هذه الـ function مفيدة عندما تريد فرز slice حسب سمة معينة لكل عنصر. في القائمة 13-7، لدينا قائمة من مثيلات `Rectangle`، ونستخدم `sort_by_key` لترتيبها حسب سمة `width` من الأدنى إلى الأعلى.

<Listing number="13-7" file-name="src/main.rs" caption="استخدام `sort_by_key` لترتيب الـ rectangles حسب الـ width">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-07/src/main.rs}}
```

</Listing>

هذا الكود يطبع:

```console
{{#include ../listings/ch13-functional-features/listing-13-07/output.txt}}
```

السبب في تعريف `sort_by_key` لأخذ Closure من نوع `FnMut` هو أنها تستدعي الـ Closure عدة مرات: مرة واحدة لكل عنصر في الـ slice. الـ Closure `|r| r.width` لا تلتقط أو تغير أو تنقل أي شيء من بيئتها، لذلك فهي تفي بمتطلبات الـ trait bound.

في المقابل، تُظهر القائمة 13-8 مثالاً لـ Closure تنفذ فقط الـ trait `FnOnce`، لأنها تنقل قيمة خارج البيئة. لن يسمح لنا الـ compiler باستخدام هذه الـ Closure مع `sort_by_key`.

<Listing number="13-8" file-name="src/main.rs" caption="محاولة استخدام Closure من نوع `FnOnce` مع `sort_by_key`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-08/src/main.rs}}
```

</Listing>

هذه طريقة مصطنعة ومعقدة (لا تعمل) لمحاولة عد عدد المرات التي تستدعي فيها `sort_by_key` الـ Closure عند فرز `list`. يحاول هذا الكود القيام بهذا العد عن طريق دفع `value`—وهي `String` من بيئة الـ Closure—إلى الـ vector `sort_operations`. تلتقط الـ Closure الـ `value` ثم تنقل الـ `value` خارج الـ Closure عن طريق نقل ownership الـ `value` إلى الـ vector `sort_operations`. يمكن استدعاء هذه الـ Closure مرة واحدة؛ محاولة استدعائها مرة ثانية لن تنجح، لأن الـ `value` لن تكون موجودة في البيئة ليتم دفعها إلى `sort_operations` مرة أخرى! لذلك، تنفذ هذه الـ Closure الـ `FnOnce` فقط. عندما نحاول compile هذا الكود، نحصل على هذا الخطأ بأن الـ `value` لا يمكن نقلها خارج الـ Closure لأن الـ Closure يجب أن تنفذ `FnMut`:

```console
{{#include ../listings/ch13-functional-features/listing-13-08/output.txt}}
```

يشير الخطأ إلى السطر في جسم الـ Closure الذي ينقل الـ `value` خارج البيئة. لإصلاح ذلك، نحتاج إلى تغيير جسم الـ Closure بحيث لا ينقل القيم خارج البيئة. الاحتفاظ بعداد في البيئة وزيادة قيمته في جسم الـ Closure هو طريقة أكثر وضوحًا لعد عدد المرات التي يتم فيها استدعاء الـ Closure. الـ Closure في القائمة 13-9 تعمل مع `sort_by_key` لأنها تلتقط فقط mutable reference لعداد `num_sort_operations` وبالتالي يمكن استدعاؤها أكثر من مرة.

<Listing number="13-9" file-name="src/main.rs" caption="يُسمح باستخدام Closure من نوع `FnMut` مع `sort_by_key`.">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-09/src/main.rs}}
```

</Listing>

الـ Fn traits مهمة عند تعريف أو استخدام functions أو أنواع تستخدم Closures. في القسم التالي، سنناقش iterators. تتطلب العديد من دوال الـ iterator وسائط Closures، لذا ضع تفاصيل الـ Closure هذه في الاعتبار بينما نواصل!

[unwrap-or-else]: ../std/option/enum.Option.html#method.unwrap_or_else
