## الـ Methods

الـ Methods تشبه الـ functions: نعلن عنها باستخدام الكلمة المفتاحية `fn` واسم، يمكن أن تحتوي على parameters وقيمة إرجاع (return value)، وتحتوي على بعض الكود الذي يتم تشغيله عند استدعاء الـ method من مكان آخر. على عكس الـ functions، يتم تعريف الـ methods ضمن سياق struct (أو enum أو trait object، والتي نغطيها في [الفصل 6][enums] و [الفصل 18][trait-objects]، على التوالي)، ويكون الـ parameter الأول لها دائمًا `self`، والذي يمثل مثيل الـ struct الذي يتم استدعاء الـ method عليه.

<!-- Old headings. Do not remove or links may break. -->

<a id="defining-methods"></a>

### بناء جملة الـ Method (Method Syntax)

دعنا نغير دالة `area` التي تحتوي على مثيل `Rectangle` كـ parameter ونجعلها بدلاً من ذلك method `area` معرفًا على struct `Rectangle`، كما هو موضح في القائمة 5-13.

<Listing number="5-13" file-name="src/main.rs" caption="تعريف method `area` على struct `Rectangle`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-13/src/main.rs}}
```

</Listing>

لتعريف الـ function ضمن سياق `Rectangle`، نبدأ كتلة `impl` (تنفيذ (implementation)) لـ `Rectangle`. كل شيء داخل كتلة `impl` هذه سيتم ربطه بنوع `Rectangle`. بعد ذلك، ننقل دالة `area` داخل الأقواس المعقوفة لـ `impl` ونغير الـ parameter الأول (والوحيد في هذه الحالة) ليكون `self` في الـ signature وفي كل مكان داخل الـ body. في `main`، حيث استدعينا دالة `area` ومررنا `rect1` كوسيط (argument)، يمكننا بدلاً من ذلك استخدام *بناء جملة الـ method* (method syntax) لاستدعاء method `area` على مثيل `Rectangle` الخاص بنا. يأتي الـ method syntax بعد مثيل: نضيف نقطة متبوعة باسم الـ method، والأقواس، وأي arguments.

في الـ signature لـ `area`، نستخدم `&self` بدلاً من `rectangle: &Rectangle`. الـ `&self` هو في الواقع اختصار لـ `self: &Self`. ضمن كتلة `impl`، النوع `Self` هو اسم مستعار (alias) للنوع الذي تنطبق عليه كتلة `impl`. يجب أن تحتوي الـ methods على parameter يسمى `self` من النوع `Self` لـ parameter الأول، لذا تسمح لك Rust باختصار ذلك باستخدام الاسم `self` فقط في موضع الـ parameter الأول. لاحظ أننا ما زلنا بحاجة إلى استخدام `&` أمام اختصار `self` للإشارة إلى أن هذا الـ method يقترض (borrows) مثيل `Self`، تمامًا كما فعلنا في `rectangle: &Rectangle`. يمكن أن تأخذ الـ methods ملكية (ownership) الـ `self`، أو تقترض الـ `self` بشكل غير قابل للتغيير (immutably)، كما فعلنا هنا، أو تقترض الـ `self` بشكل قابل للتغيير (mutably)، تمامًا كما يمكنها أي parameter آخر.

اخترنا `&self` هنا لنفس السبب الذي استخدمنا به `&Rectangle` في إصدار الـ function: لا نريد أن نأخذ الـ ownership، ونريد فقط قراءة الـ data في الـ struct، وليس الكتابة عليها. إذا أردنا تغيير المثيل الذي استدعينا الـ method عليه كجزء مما يفعله الـ method، فسنستخدم `&mut self` كـ parameter الأول. من النادر أن يكون هناك method يأخذ الـ ownership للمثيل باستخدام `self` فقط كـ parameter الأول؛ تُستخدم هذه التقنية عادةً عندما يحول الـ method الـ `self` إلى شيء آخر وتريد منع الـ caller من استخدام المثيل الأصلي بعد التحويل.

السبب الرئيسي لاستخدام الـ methods بدلاً من الـ functions، بالإضافة إلى توفير method syntax وعدم الاضطرار إلى تكرار نوع `self` في الـ signature لكل method، هو التنظيم. لقد وضعنا كل الأشياء التي يمكننا القيام بها باستخدام مثيل نوع ما في كتلة `impl` واحدة بدلاً من جعل المستخدمين المستقبليين للكود الخاص بنا يبحثون عن إمكانيات `Rectangle` في أماكن مختلفة في الـ library التي نقدمها.

لاحظ أنه يمكننا اختيار إعطاء الـ method نفس اسم أحد حقول الـ struct. على سبيل المثال، يمكننا تعريف method على `Rectangle` يسمى أيضًا `width`:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-06-method-field-interaction/src/main.rs:here}}
```

</Listing>

هنا، نختار جعل method `width` يُرجع `true` إذا كانت القيمة في حقل `width` للمثيل أكبر من `0` و `false` إذا كانت القيمة `0`: يمكننا استخدام حقل داخل method يحمل نفس الاسم لأي غرض. في `main`، عندما نتبع `rect1.width` بأقواس، تعرف Rust أننا نعني method `width`. عندما لا نستخدم الأقواس، تعرف Rust أننا نعني حقل `width`.

في كثير من الأحيان، ولكن ليس دائمًا، عندما نعطي الـ method نفس اسم الحقل، فإننا نريده فقط أن يُرجع القيمة في الحقل ولا يفعل أي شيء آخر. تسمى الـ methods مثل هذه *الـ Getters*، ولا تطبقها Rust تلقائيًا لحقول الـ struct كما تفعل بعض اللغات الأخرى. الـ Getters مفيدة لأنه يمكنك جعل الحقل خاصًا (private) ولكن الـ method عامًا (public)، وبالتالي تمكين الوصول للقراءة فقط إلى هذا الحقل كجزء من واجهة برمجة التطبيقات العامة (public API) للنوع. سنناقش ما هو public و private وكيفية تعيين حقل أو method كـ public أو private في [الفصل 7][public].

> ### أين عامل التشغيل `->`؟
>
> في C و C++، يتم استخدام عاملي تشغيل مختلفين لاستدعاء الـ methods: تستخدم `.` إذا كنت تستدعي method على الـ object مباشرة وتستخدم `->` إذا كنت تستدعي الـ method على مؤشر (pointer) إلى الـ object وتحتاج إلى dereference الـ pointer أولاً. بعبارة أخرى، إذا كان `object` مؤشرًا، فإن `object->something()` يشبه `(*object).something()`.
>
> لا تحتوي Rust على ما يعادل عامل التشغيل `->`؛ بدلاً من ذلك، تحتوي Rust على ميزة تسمى *الإشارة وإلغاء الإشارة التلقائي* (automatic referencing and dereferencing). يعد استدعاء الـ methods أحد الأماكن القليلة في Rust التي تتمتع بهذا السلوك.
>
> إليك كيفية عملها: عندما تستدعي method باستخدام `object.something()`، تضيف Rust تلقائيًا `&` أو `&mut` أو `*` بحيث يتطابق `object` مع الـ signature للـ method. بعبارة أخرى، ما يلي متماثل:
>
> <!-- CAN'T EXTRACT SEE BUG https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust
> # #[derive(Debug,Copy,Clone)]
> # struct Point {
> #     x: f64,
> #     y: f64,
> # }
> #
> # impl Point {
> #    fn distance(&self, other: &Point) -> f64 {
> #        let x_squared = f64::powi(other.x - self.x, 2);
> #        let y_squared = f64::powi(other.y - self.y, 2);
> #
> #        f64::sqrt(x_squared + y_squared)
> #    }
> # }
> # let p1 = Point { x: 0.0, y: 0.0 };
> # let p2 = Point { x: 5.0, y: 6.5 };
> p1.distance(&p2);
> (&p1).distance(&p2);
> ```
>
> يبدو الأول أكثر نظافة. يعمل سلوك الـ referencing التلقائي هذا لأن الـ methods لها مستقبل واضح - نوع `self`. بالنظر إلى مستقبل واسم الـ method، يمكن لـ Rust أن تحدد بشكل قاطع ما إذا كان الـ method يقرأ (`&self`)، أو يغير (`&mut self`)، أو يستهلك (`self`). حقيقة أن Rust تجعل الـ borrowing ضمنيًا لمستقبلات الـ method هي جزء كبير من جعل الـ ownership مريحًا (ergonomic) في الممارسة العملية.

### الـ Methods ذات الـ Parameters الإضافية

دعنا نتدرب على استخدام الـ methods من خلال تطبيق method ثانٍ على struct `Rectangle`. هذه المرة نريد أن يأخذ مثيل `Rectangle` مثيلًا آخر من `Rectangle` ويُرجع `true` إذا كان `Rectangle` الثاني يمكن أن يتناسب تمامًا داخل `self` (الـ `Rectangle` الأول)؛ وإلا، يجب أن يُرجع `false`. أي، بمجرد أن نحدد method `can_hold`، نريد أن نكون قادرين على كتابة البرنامج الموضح في القائمة 5-14.

<Listing number="5-14" file-name="src/main.rs" caption="استخدام method `can_hold` الذي لم تتم كتابته بعد">

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-14/src/main.rs}}
```

</Listing>

سيبدو الإخراج المتوقع كما يلي لأن كلا بعدي `rect2` أصغر من أبعاد `rect1`، لكن `rect3` أوسع من `rect1`:

```text
Can rect1 hold rect2? true
Can rect1 hold rect3? false
```

نعلم أننا نريد تعريف method، لذلك سيكون ضمن كتلة `impl Rectangle`. سيكون اسم الـ method هو `can_hold`، وسيأخذ اقتراضًا غير قابل للتغيير (immutable borrow) لـ `Rectangle` آخر كـ parameter. يمكننا معرفة نوع الـ parameter من خلال النظر إلى الكود الذي يستدعي الـ method: `rect1.can_hold(&rect2)` يمرر `&rect2`، وهو immutable borrow لـ `rect2`، وهو مثيل لـ `Rectangle`. هذا منطقي لأننا نحتاج فقط إلى قراءة `rect2` (بدلاً من الكتابة، مما يعني أننا سنحتاج إلى mutable borrow)، ونريد أن يحتفظ `main` بـ ownership لـ `rect2` حتى نتمكن من استخدامه مرة أخرى بعد استدعاء method `can_hold`. ستكون القيمة المرجعة لـ `can_hold` عبارة عن Boolean، وسيقوم الـ implementation بالتحقق مما إذا كان عرض وارتفاع `self` أكبر من عرض وارتفاع الـ `Rectangle` الآخر، على التوالي. دعنا نضيف method `can_hold` الجديد إلى كتلة `impl` من القائمة 5-13، الموضحة في القائمة 5-15.

<Listing number="5-15" file-name="src/main.rs" caption="تطبيق method `can_hold` على `Rectangle` الذي يأخذ مثيل `Rectangle` آخر كـ parameter">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-15/src/main.rs:here}}
```

</Listing>

عندما نقوم بتشغيل هذا الكود باستخدام دالة `main` في القائمة 5-14، سنحصل على الإخراج المطلوب. يمكن أن تأخذ الـ methods parameters متعددة نضيفها إلى الـ signature بعد parameter `self`، وتعمل هذه الـ parameters تمامًا مثل الـ parameters في الـ functions.

### الـ Functions المرتبطة (Associated Functions)

تسمى جميع الـ functions المعرفة داخل كتلة `impl` *الـ functions المرتبطة* (associated functions) لأنها مرتبطة بالنوع المسمى بعد `impl`. يمكننا تعريف associated functions لا تحتوي على `self` كـ parameter الأول لها (وبالتالي فهي ليست methods) لأنها لا تحتاج إلى مثيل من النوع للعمل معه. لقد استخدمنا بالفعل function واحدًا من هذا القبيل: دالة `String::from` المعرفة على نوع `String`.

غالبًا ما تُستخدم الـ associated functions التي ليست methods لـ الـ constructors التي ستُرجع مثيلًا جديدًا من الـ struct. غالبًا ما تسمى هذه `new`، ولكن `new` ليس اسمًا خاصًا ولم يتم بناؤه في اللغة. على سبيل المثال، يمكننا اختيار توفير associated function يسمى `square` والذي سيكون له parameter بعد واحد ويستخدم ذلك كـ عرض وارتفاع، مما يسهل إنشاء `Rectangle` مربع بدلاً من الاضطرار إلى تحديد نفس القيمة مرتين:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-03-associated-functions/src/main.rs:here}}
```

الكلمات المفتاحية `Self` في نوع الإرجاع وفي نص الـ function هي aliases للنوع الذي يظهر بعد الكلمة المفتاحية `impl`، وهو في هذه الحالة `Rectangle`.

لاستدعاء هذا الـ associated function، نستخدم بناء جملة `::` مع اسم الـ struct؛ `let sq = Rectangle::square(3);` هو مثال. هذا الـ function يتم تسميته بواسطة الـ struct: يتم استخدام بناء جملة `::` لكل من الـ associated functions والـ namespaces التي تم إنشاؤها بواسطة الـ modules. سنناقش الـ modules في [الفصل 7][modules].

### كتل `impl` المتعددة

يُسمح لكل struct أن يكون له كتل `impl` متعددة. على سبيل المثال، القائمة 5-15 مكافئة للكود الموضح في القائمة 5-16، والذي يحتوي على كل method في كتلة `impl` خاصة به.

<Listing number="5-16" caption="إعادة كتابة القائمة 5-15 باستخدام كتل `impl` متعددة">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-16/src/main.rs:here}}
```

</Listing>

لا يوجد سبب لفصل هذه الـ methods إلى كتل `impl` متعددة هنا، ولكن هذا بناء جملة صالح. سنرى حالة تكون فيها كتل `impl` المتعددة مفيدة في الفصل 10، حيث نناقش الـ generic types والـ traits.

## ملخص

تتيح لك الـ Structs إنشاء أنواع مخصصة (custom types) ذات مغزى لـ domain الخاص بك. باستخدام الـ structs، يمكنك الاحتفاظ بأجزاء الـ data المرتبطة متصلة ببعضها البعض وتسمية كل جزء لجعل الكود الخاص بك واضحًا. في كتل `impl`، يمكنك تعريف الـ functions المرتبطة بنوعك، والـ methods هي نوع من الـ associated function التي تتيح لك تحديد السلوك الذي تتمتع به مثيلات الـ structs الخاصة بك.

لكن الـ structs ليست الطريقة الوحيدة التي يمكنك من خلالها إنشاء custom types: دعنا ننتقل إلى ميزة enum في Rust لإضافة أداة أخرى إلى صندوق أدواتك.

[enums]: ch06-00-enums.html
[trait-objects]: ch18-02-trait-objects.md
[public]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html
