<!-- Old headings. Do not remove or links may break. -->

<a id="treating-smart-pointers-like-regular-references-with-the-deref-trait"></a>
<a id="treating-smart-pointers-like-regular-references-with-deref"></a>

## التعامل مع المؤشرات الذكية (Smart Pointers) كالمراجع العادية (Regular References)

يتيح لك تطبيق سمة `Deref` تخصيص سلوك *عامل إلغاء الإشارة* (dereference operator) `*` (لا يجب الخلط بينه وبين عامل الضرب أو عامل glob). من خلال تطبيق `Deref` بطريقة يمكن من خلالها التعامل مع smart pointer كـ regular reference، يمكنك كتابة كود يعمل على الـ references واستخدام هذا الكود مع الـ smart pointers أيضًا.

دعنا أولاً نلقي نظرة على كيفية عمل الـ dereference operator مع الـ regular references. بعد ذلك، سنحاول تعريف نوع مخصص يتصرف مثل `Box<T>` ونرى لماذا لا يعمل الـ dereference operator كـ reference على نوعنا المعرف حديثًا. سنستكشف كيف أن تطبيق سمة `Deref` يجعل من الممكن لـ smart pointers أن تعمل بطرق مماثلة لـ references. بعد ذلك، سننظر إلى ميزة *الإكراه على إلغاء الإشارة* (deref coercion) في Rust وكيف تتيح لنا العمل إما مع الـ references أو الـ smart pointers.

<!-- Old headings. Do not remove or links may break. -->

<a id="following-the-pointer-to-the-value-with-the-dereference-operator"></a>
<a id="following-the-pointer-to-the-value"></a>

### تتبع المرجع إلى القيمة

الـ regular reference هو نوع من المؤشرات (pointer)، وإحدى طرق التفكير في الـ pointer هي أنه سهم يشير إلى قيمة مخزنة في مكان آخر. في القائمة 15-6، ننشئ reference إلى قيمة `i32` ثم نستخدم الـ dereference operator لتتبع الـ reference إلى القيمة.

<Listing number="15-6" file-name="src/main.rs" caption="استخدام عامل إلغاء الإشارة لتتبع reference إلى قيمة `i32`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-06/src/main.rs}}
```

</Listing>

يحتوي المتغير `x` على قيمة `i32` وهي `5`. نضبط `y` ليكون مساويًا لـ reference إلى `x`. يمكننا التأكيد على أن `x` يساوي `5`. ومع ذلك، إذا أردنا إجراء تأكيد حول القيمة في `y`، يجب علينا استخدام `*y` لتتبع الـ reference إلى القيمة التي يشير إليها (وبالتالي، *إلغاء الإشارة* (dereference)) حتى يتمكن الـ compiler من مقارنة القيمة الفعلية. بمجرد أن نقوم بـ dereference لـ `y`، يمكننا الوصول إلى القيمة الصحيحة التي يشير إليها `y` والتي يمكننا مقارنتها بـ `5`.

إذا حاولنا كتابة `assert_eq!(5, y);` بدلاً من ذلك، فسنحصل على خطأ الـ compilation هذا:

```console
{{#include ../listings/ch15-smart-pointers/output-only-01-comparing-to-reference/output.txt}}
```

لا يُسمح بمقارنة رقم و reference إلى رقم لأنهما نوعان مختلفان. يجب علينا استخدام الـ dereference operator لتتبع الـ reference إلى القيمة التي يشير إليها.

### استخدام `Box<T>` كـ Reference

يمكننا إعادة كتابة الكود في القائمة 15-6 لاستخدام `Box<T>` بدلاً من reference؛ يعمل الـ dereference operator المستخدم على `Box<T>` في القائمة 15-7 بنفس طريقة عمل الـ dereference operator المستخدم على الـ reference في القائمة 15-6.

<Listing number="15-7" file-name="src/main.rs" caption="استخدام عامل إلغاء الإشارة على `Box<i32>`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-07/src/main.rs}}
```

</Listing>

الفرق الرئيسي بين القائمة 15-7 والقائمة 15-6 هو أننا هنا نضبط `y` ليكون مثيلًا لـ box يشير إلى قيمة منسوخة من `x` بدلاً من reference يشير إلى قيمة `x`. في التأكيد الأخير، يمكننا استخدام الـ dereference operator لتتبع مؤشر الـ box بنفس الطريقة التي فعلناها عندما كان `y` reference. بعد ذلك، سنستكشف ما هو خاص بـ `Box<T>` الذي يمكننا من استخدام الـ dereference operator عن طريق تعريف نوع الـ box الخاص بنا.

### تعريف الـ Smart Pointer الخاص بنا

دعنا نبني نوع مغلف (wrapper type) مشابه لـ `Box<T>` الذي توفره الـ standard library لتجربة كيف تتصرف أنواع الـ smart pointer بشكل مختلف عن الـ references افتراضيًا. بعد ذلك، سننظر في كيفية إضافة القدرة على استخدام الـ dereference operator.

> ملاحظة: هناك فرق كبير واحد بين نوع `MyBox<T>` الذي سنبنيه الآن و `Box<T>` الحقيقي: لن يقوم إصدارنا بتخزين بياناته على الكومة (heap). نحن نركز هذا المثال على `Deref`، لذا فإن مكان تخزين البيانات فعليًا أقل أهمية من سلوك الـ pointer.

يتم تعريف النوع `Box<T>` في النهاية على أنه struct tuple بعنصر واحد، لذا تحدد القائمة 15-8 نوع `MyBox<T>` بنفس الطريقة. سنقوم أيضًا بتعريف دالة `new` لتتناسب مع دالة `new` المعرفة على `Box<T>`.

<Listing number="15-8" file-name="src/main.rs" caption="تعريف نوع `MyBox<T>`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-08/src/main.rs:here}}
```

</Listing>

نحدد struct يسمى `MyBox` ونعلن عن generic parameter `T` لأننا نريد أن يحتوي نوعنا على قيم من أي نوع. نوع `MyBox` هو tuple struct بعنصر واحد من النوع `T`. تأخذ الدالة `MyBox::new` parameter واحدًا من النوع `T` وتُرجع مثيل `MyBox` يحتوي على القيمة التي تم تمريرها.

دعنا نحاول إضافة الدالة `main` في القائمة 15-7 إلى القائمة 15-8 وتغييرها لاستخدام نوع `MyBox<T>` الذي عرفناه بدلاً من `Box<T>`. لن يتم تجميع الكود في القائمة 15-9، لأن Rust لا تعرف كيفية dereference لـ `MyBox`.

<Listing number="15-9" file-name="src/main.rs" caption="محاولة استخدام `MyBox<T>` بنفس الطريقة التي استخدمنا بها الـ references و `Box<T>`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-09/src/main.rs:here}}
```

</Listing>

إليك خطأ الـ compilation الناتج:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-09/output.txt}}
```

لا يمكن إلغاء الإشارة إلى نوع `MyBox<T>` الخاص بنا لأننا لم نطبق هذه القدرة على نوعنا. لتمكين إلغاء الإشارة باستخدام الـ operator `*`، نقوم بتطبيق سمة `Deref`.

<!-- Old headings. Do not remove or links may break. -->

<a id="treating-a-type-like-a-reference-by-implementing-the-deref-trait"></a>

### تطبيق سمة `Deref`

كما نوقش في ["تطبيق سمة على نوع"][impl-trait] في الفصل 10، لتطبيق سمة (trait) نحتاج إلى توفير تطبيقات لـ methods السمة المطلوبة. تتطلب سمة `Deref`، التي توفرها الـ standard library، منا تطبيق method واحد يسمى `deref` يقترض `self` ويُرجع reference إلى البيانات الداخلية. تحتوي القائمة 15-10 على تطبيق لـ `Deref` لإضافته إلى تعريف `MyBox<T>`.

<Listing number="15-10" file-name="src/main.rs" caption="تطبيق `Deref` على `MyBox<T>`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-10/src/main.rs:here}}
```

</Listing>

يحدد بناء الجملة `type Target = T;` نوعًا مرتبطًا (associated type) لـ سمة `Deref` لاستخدامه. الـ associated types هي طريقة مختلفة قليلاً للإعلان عن generic parameter، ولكن لا داعي للقلق بشأنها الآن؛ سنتناولها بمزيد من التفصيل في الفصل 20.

نملأ نص method `deref` بـ `&self.0` بحيث يُرجع `deref` reference إلى القيمة التي نريد الوصول إليها باستخدام الـ operator `*`؛ تذكر من ["إنشاء أنواع مختلفة باستخدام Structs Tuple"][tuple-structs] في الفصل 5 أن `.0` يصل إلى القيمة الأولى في tuple struct. الآن يتم تجميع الدالة `main` في القائمة 15-9 التي تستدعي `*` على قيمة `MyBox<T>`، وتمر التأكيدات!

بدون سمة `Deref`، يمكن لـ compiler فقط dereference لـ references `&`. يمنح method `deref` الـ compiler القدرة على أخذ قيمة من أي نوع يطبق `Deref` واستدعاء method `deref` للحصول على reference يعرف كيفية dereference له.

عندما أدخلنا `*y` في القائمة 15-9، قامت Rust فعليًا بتشغيل هذا الكود خلف الكواليس:

```rust,ignore
*(y.deref())
```

تستبدل Rust الـ operator `*` باستدعاء لـ method `deref` ثم dereference عادي حتى لا نضطر إلى التفكير فيما إذا كنا بحاجة إلى استدعاء method `deref` أم لا. تتيح لنا ميزة Rust هذه كتابة كود يعمل بشكل متطابق سواء كان لدينا regular reference أو نوع يطبق `Deref`.

يرجع سبب إرجاع method `deref` لـ reference إلى قيمة، وأن الـ dereference العادي خارج الأقواس في `*(y.deref())` لا يزال ضروريًا، إلى نظام الملكية (ownership system). إذا كان method `deref` يُرجع القيمة مباشرة بدلاً من reference إلى القيمة، فسيتم نقل القيمة خارج `self`. لا نريد أن نأخذ ownership للقيمة الداخلية داخل `MyBox<T>` في هذه الحالة أو في معظم الحالات التي نستخدم فيها الـ dereference operator.

لاحظ أنه يتم استبدال الـ operator `*` باستدعاء لـ method `deref` ثم استدعاء لـ operator `*` مرة واحدة فقط، في كل مرة نستخدم فيها `*` في الكود الخاص بنا. نظرًا لأن استبدال الـ operator `*` لا يتكرر إلى ما لا نهاية، فإننا ننتهي ببيانات من النوع `i32`، والتي تتطابق مع `5` في `assert_eq!` في القائمة 15-9.

<!-- Old headings. Do not remove or links may break. -->

<a id="implicit-deref-coercions-with-functions-and-methods"></a>
<a id="using-deref-coercions-in-functions-and-methods"></a>

### استخدام Deref Coercion في الدوال والـ Methods

*الإكراه على إلغاء الإشارة* (Deref coercion) يحول reference إلى نوع يطبق سمة `Deref` إلى reference إلى نوع آخر. على سبيل المثال، يمكن لـ deref coercion تحويل `&String` إلى `&str` لأن `String` تطبق سمة `Deref` بحيث تُرجع `&str`. الـ Deref coercion هي ميزة راحة تقوم بها Rust على الوسائط (arguments) لـ functions والـ methods، وتعمل فقط على الـ types التي تطبق سمة `Deref`. يحدث تلقائيًا عندما نمرر reference إلى قيمة نوع معين كوسيط لـ function أو method لا يتطابق مع نوع الـ parameter في تعريف الـ function أو method. تحول سلسلة من الاستدعاءات لـ method `deref` النوع الذي قدمناه إلى النوع الذي يحتاجه الـ parameter.

تمت إضافة الـ Deref coercion إلى Rust حتى لا يضطر المبرمجون الذين يكتبون استدعاءات الـ function والـ method إلى إضافة العديد من الـ references و dereferences الصريحة باستخدام `&` و `*`. تتيح لنا ميزة deref coercion أيضًا كتابة المزيد من الكود الذي يمكن أن يعمل إما لـ references أو لـ smart pointers.

لرؤية deref coercion أثناء العمل، دعنا نستخدم نوع `MyBox<T>` الذي عرفناه في القائمة 15-8 بالإضافة إلى تطبيق `Deref` الذي أضفناه في القائمة 15-10. تُظهر القائمة 15-11 تعريف function يحتوي على string slice parameter.

<Listing number="15-11" file-name="src/main.rs" caption="دالة `hello` تحتوي على الـ parameter `name` من النوع `&str`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-11/src/main.rs:here}}
```

</Listing>

يمكننا استدعاء الدالة `hello` باستخدام string slice كوسيط، مثل `hello("Rust");`، على سبيل المثال. تجعل الـ Deref coercion من الممكن استدعاء `hello` بـ reference إلى قيمة من النوع `MyBox<String>`، كما هو موضح في القائمة 15-12.

<Listing number="15-12" file-name="src/main.rs" caption="استدعاء `hello` بـ reference إلى قيمة `MyBox<String>`، والذي يعمل بسبب deref coercion">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-12/src/main.rs:here}}
```

</Listing>

هنا نستدعي الدالة `hello` بالوسيط `&m`، وهو reference إلى قيمة `MyBox<String>`. نظرًا لأننا طبقنا سمة `Deref` على `MyBox<T>` في القائمة 15-10، يمكن لـ Rust تحويل `&MyBox<String>` إلى `&String` عن طريق استدعاء `deref`. توفر الـ standard library تطبيقًا لـ `Deref` على `String` يُرجع string slice، وهذا موجود في وثائق API لـ `Deref`. تستدعي Rust `deref` مرة أخرى لتحويل `&String` إلى `&str`، والذي يتطابق مع تعريف الدالة `hello`.

إذا لم تطبق Rust الـ deref coercion، فسيتعين علينا كتابة الكود في القائمة 15-13 بدلاً من الكود في القائمة 15-12 لاستدعاء `hello` بقيمة من النوع `&MyBox<String>`.

<Listing number="15-13" file-name="src/main.rs" caption="الكود الذي سيتعين علينا كتابته إذا لم يكن لدى Rust deref coercion">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-13/src/main.rs:here}}
```

</Listing>

هذا الكود بدون deref coercions أصعب في القراءة والكتابة والفهم مع كل هذه الرموز المعنية. تسمح الـ Deref coercion لـ Rust بالتعامل مع هذه التحويلات تلقائيًا نيابة عنا.

عندما يتم تعريف سمة `Deref` للأنواع المعنية، ستحلل Rust الـ types وتستخدم `Deref::deref` عدة مرات حسب الضرورة للحصول على reference يتطابق مع نوع الـ parameter. يتم حل عدد المرات التي يجب فيها إدراج `Deref::deref` في الـ compile time، لذلك لا توجد عقوبة في وقت التشغيل للاستفادة من deref coercion!

<!-- Old headings. Do not remove or links may break. -->

<a id="how-deref-coercion-interacts-with-mutability"></a>

### التعامل مع Deref Coercion باستخدام المراجع القابلة للتغيير (Mutable References)

على غرار كيفية استخدامك لسمة `Deref` لتجاوز الـ operator `*` على الـ immutable references، يمكنك استخدام سمة `DerefMut` لتجاوز الـ operator `*` على الـ mutable references.

تقوم Rust بـ deref coercion عندما تجد الـ types وتطبيقات الـ trait في ثلاث حالات:

1. من `&T` إلى `&U` عندما `T: Deref<Target=U>`
2. من `&mut T` إلى `&mut U` عندما `T: DerefMut<Target=U>`
3. من `&mut T` إلى `&U` عندما `T: Deref<Target=U>`

الحالتان الأوليان متماثلتان باستثناء أن الثانية تطبق القابلية للتغيير (mutability). تنص الحالة الأولى على أنه إذا كان لديك `&T`، وطبق `T` سمة `Deref` على نوع ما `U`، يمكنك الحصول على `&U` بشفافية. تنص الحالة الثانية على أن نفس الـ deref coercion يحدث لـ mutable references.

الحالة الثالثة أكثر تعقيدًا: ستقوم Rust أيضًا بـ coerce لـ mutable reference إلى immutable reference. لكن العكس *غير* ممكن: لن يتم أبدًا coerce لـ immutable references إلى mutable references. بسبب قواعد الاقتراض (borrowing rules)، إذا كان لديك mutable reference، فيجب أن يكون هذا الـ mutable reference هو الـ reference الوحيد لتلك البيانات (وإلا فلن يتم تجميع البرنامج). لن يؤدي تحويل mutable reference واحد إلى immutable reference واحد إلى كسر قواعد الاقتراض أبدًا. سيتطلب تحويل immutable reference إلى mutable reference أن يكون الـ immutable reference الأولي هو الـ immutable reference الوحيد لتلك البيانات، لكن قواعد الاقتراض لا تضمن ذلك. لذلك، لا يمكن لـ Rust افتراض أن تحويل immutable reference إلى mutable reference ممكن.

[impl-trait]: ch10-02-traits.html#implementing-a-trait-on-a-type
[tuple-structs]: ch05-01-defining-structs.html#creating-different-types-with-tuple-structs
