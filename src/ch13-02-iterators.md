## معالجة سلسلة من العناصر باستخدام المكررات (Processing a Series of Items with Iterators)

يسمح لك نمط المكرر (iterator pattern) بأداء بعض المهام على تسلسل من العناصر بالدور. المكرر (iterator) مسؤول عن منطق التكرار (iterating) عبر كل عنصر وتحديد متى ينتهي التسلسل. عندما تستخدم iterators، ليس عليك إعادة تنفيذ ذلك المنطق بنفسك.

في Rust، تكون الـ iterators _كسولة_ (lazy)، مما يعني أنه ليس لها أي تأثير حتى تستدعي طرقاً (methods) تستهلك (consume) الـ iterator لاستخدامه بالكامل. على سبيل المثال، الكود في القائمة 13-10 ينشئ iterator عبر العناصر الموجودة في المتجه (vector) `v1` عن طريق استدعاء طريقة `iter` المعرفة على `Vec<T>`. هذا الكود بحد ذاته لا يفعل أي شيء مفيد.

<Listing number="13-10" file-name="src/main.rs" caption="Creating an iterator">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-10/src/main.rs:here}}
```

</Listing>

يتم تخزين الـ iterator في المتغير `v1_iter`. بمجرد إنشاء iterator، يمكننا استخدامه بطرق متنوعة. في القائمة 3-5، قمنا بالتكرار عبر مصفوفة (array) باستخدام حلقة `for` لتنفيذ بعض الكود على كل عنصر من عناصرها. خلف الكواليس، أدى هذا ضمناً إلى إنشاء iterator ثم استهلاكه، لكننا أغفلنا كيفية عمل ذلك بالضبط حتى الآن.

في المثال في القائمة 13-11، نفصل إنشاء الـ iterator عن استخدامه في حلقة `for`. عندما يتم استدعاء حلقة `for` باستخدام الـ iterator في `v1_iter` ، يتم استخدام كل عنصر في الـ iterator في دورة واحدة من الحلقة، والتي تطبع كل قيمة.

<Listing number="13-11" file-name="src/main.rs" caption="Using an iterator in a `for` loop">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-11/src/main.rs:here}}
```

</Listing>

في اللغات التي لا توفر iterators في مكتباتها القياسية، من المحتمل أن تكتب هذه الوظيفة نفسها عن طريق بدء متغير عند الفهرس (index) 0، واستخدام ذلك المتغير للفهرسة في الـ vector للحصول على قيمة، وزيادة قيمة المتغير في حلقة حتى يصل إلى العدد الإجمالي للعناصر في الـ vector.

تتعامل الـ iterators مع كل هذا المنطق نيابة عنك، مما يقلل من الكود المتكرر الذي قد تخطئ فيه. تمنحك الـ iterators مرونة أكبر لاستخدام نفس المنطق مع أنواع مختلفة من التسلسلات، وليس فقط هياكل البيانات التي يمكنك الفهرسة فيها، مثل vectors. دعونا نفحص كيف تفعل الـ iterators ذلك.

### سمة المكرر وطريقة التالي (The `Iterator` Trait and the `next` Method)

تنفذ جميع الـ iterators سمة (trait) تسمى `Iterator` معرفة في المكتبة القياسية. يبدو تعريف الـ trait هكذا:

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // methods with default implementations elided
}
```

لاحظ أن هذا التعريف يستخدم بعض الصيغ (syntax) الجديدة: `type Item` و `Self::Item` ، والتي تعرف نوعاً مرتبطاً (associated type) بهذا الـ trait. سنتحدث عن associated types بعمق في الفصل العشرين. في الوقت الحالي، كل ما تحتاج لمعرفته هو أن هذا الكود يقول إن تنفيذ `Iterator` trait يتطلب منك أيضاً تعريف نوع `Item` ، ويتم استخدام نوع `Item` هذا في نوع الإرجاع لطريقة `next`. بعبارة أخرى، سيكون نوع `Item` هو النوع الذي يتم إرجاعه من الـ iterator.

تتطلب `Iterator` trait من المنفذين تعريف طريقة واحدة فقط: طريقة `next` ، والتي تعيد عنصراً واحداً من الـ iterator في كل مرة، مغلفاً في `Some` ، وعندما ينتهي التكرار، تعيد `None`.

يمكننا استدعاء طريقة `next` على الـ iterators مباشرة؛ توضح القائمة 13-12 القيم التي يتم إرجاعها من الاستدعاءات المتكررة لـ `next` على الـ iterator المنشأ من الـ vector.

<Listing number="13-12" file-name="src/lib.rs" caption="Calling the `next` method on an iterator">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-12/src/lib.rs:here}}
```

</Listing>

لاحظ أننا احتجنا إلى جعل `v1_iter` قابلاً للتغيير (mutable): استدعاء طريقة `next` على iterator يغير الحالة الداخلية التي يستخدمها الـ iterator لتتبع مكانه في التسلسل. بعبارة أخرى، هذا الكود _يستهلك_ (consumes)، أو يستنفد، الـ iterator. كل استدعاء لـ `next` يأكل عنصراً من الـ iterator. لم نكن بحاجة إلى جعل `v1_iter` mutable عندما استخدمنا حلقة `for` ، لأن الحلقة أخذت ملكية (ownership) `v1_iter` وجعلته mutable خلف الكواليس.

لاحظ أيضاً أن القيم التي نحصل عليها من الاستدعاءات لـ `next` هي مراجع غير قابلة للتغيير (immutable references) للقيم الموجودة في الـ vector. تنتج طريقة `iter` مكرراً عبر immutable references. إذا أردنا إنشاء iterator يأخذ ownership لـ `v1` ويعيد قيم مملوكة، يمكننا استدعاء `into_iter` بدلاً من `iter`. وبالمثل، إذا أردنا التكرار عبر مراجع قابلة للتغيير (mutable references)، يمكننا استدعاء `iter_mut` بدلاً من `iter`.

### الطرق التي تستهلك المكرر (Methods That Consume the Iterator)

تمتلك `Iterator` trait عدداً من الطرق المختلفة مع تنفيذات افتراضية توفرها المكتبة القياسية؛ يمكنك التعرف على هذه الطرق من خلال النظر في توثيق API للمكتبة القياسية لـ `Iterator` trait. تستدعي بعض هذه الطرق طريقة `next` في تعريفها، وهذا هو السبب في أنك مطالب بتنفيذ طريقة `next` عند تنفيذ `Iterator` trait.

تسمى الطرق التي تستدعي `next` بـ _محولات الاستهلاك_ (consuming adapters) لأن استدعاءها يستنفد الـ iterator. أحد الأمثلة هو طريقة `sum` ، التي تأخذ ownership للـ iterator وتكرر عبر العناصر من خلال استدعاء `next` بشكل متكرر، وبالتالي تستهلك الـ iterator. وبينما تكرر، تضيف كل عنصر إلى إجمالي جاري وتعيد الإجمالي عند اكتمال التكرار. تحتوي القائمة 13-13 على اختبار يوضح استخدام طريقة `sum`.

<Listing number="13-13" file-name="src/lib.rs" caption="Calling the `sum` method to get the total of all items in the iterator">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-13/src/lib.rs:here}}
```

</Listing>

لا يُسمح لنا باستخدام `v1_iter` بعد استدعاء `sum` ، لأن `sum` تأخذ ownership للـ iterator الذي نستدعيها عليه.

### الطرق التي تنتج مكررات أخرى (Methods That Produce Other Iterators)

_محولات المكرر_ (Iterator adapters) هي طرق معرفة على `Iterator` trait لا تستهلك الـ iterator. بدلاً من ذلك، فإنها تنتج iterators مختلفة عن طريق تغيير بعض جوانب الـ iterator الأصلي.

توضح القائمة 13-14 مثالاً على استدعاء طريقة محول المكرر `map` ، والتي تأخذ إغلاقاً (closure) لاستدعائه على كل عنصر أثناء التكرار عبر العناصر. تعيد طريقة `map` مكرراً جديداً ينتج العناصر المعدلة. الـ closure هنا ينشئ iterator جديداً يتم فيه زيادة كل عنصر من الـ vector بمقدار 1.

<Listing number="13-14" file-name="src/main.rs" caption="Calling the iterator adapter `map` to create a new iterator">

```rust,not_desired_behavior
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-14/src/main.rs:here}}
```

</Listing>

ومع ذلك، ينتج هذا الكود تحذيراً:

```console
{{#include ../listings/ch13-functional-features/listing-13-14/output.txt}}
```

الكود في القائمة 13-14 لا يفعل أي شيء؛ الـ closure الذي حددناه لا يتم استدعاؤه أبداً. يذكرنا التحذير بالسبب: iterator adapters كسولة، ونحن بحاجة إلى استهلاك الـ iterator هنا.

لإصلاح هذا التحذير واستهلاك الـ iterator، سنستخدم طريقة `collect` ، التي استخدمناها مع `env::args` في القائمة 12-1. تستهلك هذه الطريقة الـ iterator وتجمع القيم الناتجة في نوع بيانات تجميعي (collection).

في القائمة 13-15، نجمع نتائج التكرار عبر الـ iterator الذي يتم إرجاعه من الاستدعاء لـ `map` في vector. سينتهي هذا الـ vector باحتواء كل عنصر من الـ vector الأصلي، مزيداً بمقدار 1.

<Listing number="13-15" file-name="src/main.rs" caption="Calling the `map` method to create a new iterator, and then calling the `collect` method to consume the new iterator and create a vector">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-15/src/main.rs:here}}
```

</Listing>

لأن `map` تأخذ closure، يمكننا تحديد أي عملية نريد القيام بها على كل عنصر. هذا مثال رائع على كيفية سماح closures لك بتخصيص بعض السلوك مع إعادة استخدام سلوك التكرار الذي توفره `Iterator` trait.

يمكنك ربط استدعاءات متعددة لـ iterator adapters لأداء إجراءات معقدة بطريقة مقروءة. ولكن لأن جميع الـ iterators كسولة، يجب عليك استدعاء إحدى طرق consuming adapter للحصول على نتائج من الاستدعاءات لـ iterator adapters.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-closures-that-capture-their-environment"></a>

### الإغلاقات التي تلتقط بيئتها (Closures That Capture Their Environment)

تأخذ العديد من iterator adapters إغلاقات كـ arguments، وعادة ما تكون الـ closures التي سنحددها كـ arguments لـ iterator adapters هي closures تلتقط بيئتها.

لهذا المثال، سنستخدم طريقة `filter` التي تأخذ closure. يحصل الـ closure على عنصر من الـ iterator ويعيد `bool`. إذا أعاد الـ closure القيمة `true` ، فسيتم تضمين القيمة في التكرار الناتج عن `filter`. إذا أعاد الـ closure القيمة `false` ، فلن يتم تضمين القيمة.

في القائمة 13-16، نستخدم `filter` مع closure يلتقط المتغير `shoe_size` من بيئته للتكرار عبر مجموعة من مثيلات (instances) هيكل `Shoe`. سيعيد فقط الأحذية التي هي بالحجم المحدد.

<Listing number="13-16" file-name="src/lib.rs" caption="Using the `filter` method with a closure that captures `shoe_size`">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-16/src/lib.rs}}
```

</Listing>

تأخذ دالة `shoes_in_size` ملكية vector من الأحذية وحجم حذاء كـ parameters. وتعيد vector يحتوي فقط على الأحذية بالحجم المحدد.

في جسم `shoes_in_size` ، نستدعي `into_iter` لإنشاء iterator يأخذ ownership للـ vector. ثم، نستدعي `filter` لتكييف ذلك الـ iterator إلى iterator جديد يحتوي فقط على العناصر التي يعيد الـ closure لها القيمة `true`.

يلتقط الـ closure الـ parameter `shoe_size` من البيئة ويقارن القيمة بحجم كل حذاء، محتفظاً فقط بالأحذية بالحجم المحدد. أخيراً، يؤدي استدعاء `collect` إلى جمع القيم التي أرجعها الـ iterator المكيف في vector يتم إرجاعه بواسطة الدالة.

يظهر الاختبار أنه عندما نستدعي `shoes_in_size` ، نحصل فقط على الأحذية التي لها نفس الحجم مثل القيمة التي حددناها.
```
