## (السمات المتقدمة) Advanced Traits

لقد غطينا `Traits` لأول مرة في قسم ["تعريف السلوك المشترك باستخدام السمات"][traits] في الفصل العاشر، لكننا لم نناقش التفاصيل الأكثر تقدماً. الآن بعد أن عرفت المزيد عن Rust، يمكننا الدخول في التفاصيل الدقيقة.

### تعريف السمات باستخدام الأنواع المرتبطة (Defining Traits with Associated Types)

تربط (الأنواع المرتبطة) `Associated types` بين (نائب نوع) `type placeholder` و (سمة) `trait` بحيث يمكن لتعريفات (دوال السمة) `trait methods` استخدام هذه الأنواع النائبة في (تواقيعها) `signatures`. سيقوم (منفذ السمة) `implementor` بتحديد (النوع الملموس) `concrete type` الذي سيتم استخدامه بدلاً من النوع النائب لتنفيذ معين. بهذه الطريقة، يمكننا تعريف `trait` يستخدم بعض الأنواع دون الحاجة إلى معرفة ماهية تلك الأنواع بالضبط حتى يتم تنفيذ `trait`.

لقد وصفنا معظم الميزات المتقدمة في هذا الفصل بأنها نادراً ما تكون مطلوبة. تقع `Associated types` في مكان ما في المنتصف: فهي تُستخدم بشكل أقل تكراراً من الميزات الموضحة في بقية الكتاب ولكن بشكل أكثر شيوعاً من العديد من الميزات الأخرى التي تمت مناقشتها في هذا الفصل.

أحد الأمثلة على `trait` مع `associated type` هو `Iterator` الذي توفره المكتبة القياسية. يسمى `associated type` بـ `Item` ويمثل نوع القيم التي يتنقل عبرها النوع الذي ينفذ `Iterator`. تعريف `Iterator` هو كما هو موضح في القائمة 20-13.

<Listing number="20-13" caption="تعريف سمة `Iterator` التي تحتوي على نوع مرتبط `Item` ">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-13/src/lib.rs}}
```

</Listing>

النوع `Item` هو `placeholder` (نائب)، ويوضح تعريف دالة `next` أنها ستعيد قيمًا من النوع `Option<Self::Item>`. سيقوم منفذو `Iterator` بتحديد `concrete type` لـ `Item` ، وستعيد دالة `next` نوع `Option` يحتوي على قيمة من ذلك `concrete type`.

قد تبدو `Associated types` مفهوماً مشابهاً لـ (الأنواع العامة) `generics` ، حيث تسمح لنا الأخيرة بتعريف دالة دون تحديد الأنواع التي يمكنها التعامل معها. لفحص الفرق بين المفهومين، سننظر في تنفيذ `Iterator` على نوع يسمى `Counter` يحدد أن نوع `Item` هو `u32`:

<Listing file-name="src/lib.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-22-iterator-on-counter/src/lib.rs:ch19}}
```

</Listing>

تبدو هذه الصيغة قابلة للمقارنة مع `generics`. لذا، لماذا لا نعرف `Iterator` باستخدام `generics` فقط، كما هو موضح في القائمة 20-14؟

<Listing number="20-14" caption="تعريف افتراضي لسمة `Iterator` باستخدام الأنواع العامة">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-14/src/lib.rs}}
```

</Listing>

الفرق هو أنه عند استخدام `generics` ، كما في القائمة 20-14، يجب علينا (توضيح) `annotate` الأنواع في كل تنفيذ؛ ولأننا نستطيع أيضاً تنفيذ `Iterator<String> for Counter` أو أي نوع آخر، فقد يكون لدينا تطبيقات متعددة لـ `Iterator` لـ `Counter`. بعبارة أخرى، عندما يحتوي `trait` على (معلمة عامة) `generic parameter` ، يمكن تنفيذه لنوع ما عدة مرات، مع تغيير `concrete types` لـ `generic type parameters` في كل مرة. عندما نستخدم دالة `next` على `Counter` ، سيتعين علينا تقديم (توضيحات النوع) `type annotations` للإشارة إلى أي تنفيذ لـ `Iterator` نريد استخدامه.

مع `Associated types` ، لا نحتاج إلى `annotate` الأنواع، لأننا لا نستطيع تنفيذ `trait` على نوع ما عدة مرات. في القائمة 20-13 مع التعريف الذي يستخدم `associated types` ، يمكننا اختيار نوع `Item` مرة واحدة فقط لأنه لا يمكن أن يكون هناك سوى `impl Iterator for Counter` واحد. ليس علينا تحديد أننا نريد (مكررًا) `iterator` لقيم `u32` في كل مكان نستدعي فيه `next` على `Counter`.

تصبح `Associated types` أيضاً جزءاً من (عقد السمة) `trait’s contract`: يجب على منفذي `trait` توفير نوع ليحل محل `associated type placeholder`. غالباً ما يكون لـ `Associated types` اسم يصف كيفية استخدام النوع، ويعد توثيق `associated type` في توثيق (واجهة برمجة التطبيقات) `API` ممارسة جيدة.

### استخدام المعلمات العامة الافتراضية والتحميل الزائد للمعاملات (Using Default Generic Parameters and Operator Overloading)

عندما نستخدم `generic type parameters` ، يمكننا تحديد `concrete type` افتراضي للنوع العام. هذا يلغي حاجة منفذي `trait` لتحديد `concrete type` إذا كان النوع الافتراضي يعمل. يمكنك تحديد نوع افتراضي عند التصريح عن نوع عام باستخدام صيغة `<PlaceholderType=ConcreteType>`.

مثال رائع على موقف تكون فيه هذه التقنية مفيدة هو (التحميل الزائد للمعاملات) `operator overloading` ، حيث تقوم بتخصيص سلوك (معامل) `operator` (مثل `+`) في مواقف معينة.

لا تسمح لك Rust بإنشاء `operators` خاصة بك أو تحميل `operators` عشوائية بشكل زائد. ولكن يمكنك تحميل العمليات والسمات المقابلة المدرجة في `std::ops` عن طريق تنفيذ `traits` المرتبطة بـ `operator`. على سبيل المثال، في القائمة 20-15، نقوم بتحميل المعامل `+` بشكل زائد لجمع مثيلين من `Point` معاً. نقوم بذلك عن طريق تنفيذ سمة `Add` على (هيكل) `struct` باسم `Point`.

<Listing number="20-15" file-name="src/main.rs" caption="تنفيذ سمة `Add` لتحميل المعامل `+` بشكل زائد لمثيلات `Point` ">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-15/src/main.rs}}
```

</Listing>

تقوم دالة `add` بجمع قيم `x` لمثيلين من `Point` وقيم `y` لمثيلين من `Point` لإنشاء `Point` جديد. تحتوي سمة `Add` على `associated type` يسمى `Output` يحدد النوع الذي تعيده دالة `add`.

`generic type` الافتراضي في هذا الكود موجود داخل سمة `Add`. إليك تعريفها:

```rust
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
```

يجب أن يبدو هذا الكود مألوفاً بشكل عام: `trait` مع دالة واحدة و `associated type`. الجزء الجديد هو `Rhs=Self`: تسمى هذه الصيغة (معلمات النوع الافتراضية) `default type parameters`. تحدد معلمة النوع العام `Rhs` (اختصار لـ "الجانب الأيمن" `right-hand side`) نوع معلمة `rhs` في دالة `add`. إذا لم نحدد `concrete type` لـ `Rhs` عندما ننفذ سمة `Add` ، فسيتم تعيين نوع `Rhs` افتراضياً إلى `Self` ، والذي سيكون النوع الذي ننفذ `Add` عليه.

عندما نفذنا `Add` لـ `Point` ، استخدمنا الافتراضي لـ `Rhs` لأننا أردنا جمع مثيلين من `Point`. دعنا ننظر في مثال لتنفيذ سمة `Add` حيث نريد تخصيص نوع `Rhs` بدلاً من استخدام الافتراضي.

لدينا هيكلان، `Millimeters` و `Meters` ، يحملان قيمًا بوحدات مختلفة. يُعرف هذا التغليف الرقيق لنوع موجود في `struct` آخر باسم (نمط النوع الجديد) `newtype pattern` ، والذي نصفه بمزيد من التفصيل في قسم ["تنفيذ السمات الخارجية باستخدام نمط النوع الجديد"][newtype]. نريد جمع قيم بالمليمترات إلى قيم بالأمتار وجعل تنفيذ `Add` يقوم بالتحويل بشكل صحيح. يمكننا تنفيذ `Add` لـ `Millimeters` مع `Meters` كـ `Rhs` ، كما هو موضح في القائمة 20-16.

<Listing number="20-16" file-name="src/lib.rs" caption="تنفيذ سمة `Add` على `Millimeters` لجمع `Millimeters` و `Meters` ">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-16/src/lib.rs}}
```

</Listing>

لجمع `Millimeters` و `Meters` ، نحدد `impl Add<Meters>` لتعيين قيمة معلمة النوع `Rhs` بدلاً من استخدام الافتراضي `Self`.

ستستخدم `default type parameters` بطريقتين رئيسيتين:

1. لتوسيع نوع دون كسر الكود الموجود.
2. للسماح بالتخصيص في حالات محددة لن يحتاجها معظم المستخدمين.

سمة `Add` في المكتبة القياسية هي مثال على الغرض الثاني: عادةً، ستجمع نوعين متشابهين، لكن سمة `Add` توفر القدرة على التخصيص بما يتجاوز ذلك. استخدام `default type parameter` في تعريف سمة `Add` يعني أنك لست مضطراً لتحديد المعلمة الإضافية معظم الوقت. بعبارة أخرى، لا توجد حاجة لبعض (الأكواد المتكررة) `boilerplate` للتنفيذ، مما يسهل استخدام `trait`.

الغرض الأول مشابه للثاني ولكن بالعكس: إذا كنت تريد إضافة معلمة نوع إلى `trait` موجود، يمكنك إعطاؤها قيمة افتراضية للسماح بتوسيع وظائف `trait` دون كسر كود التنفيذ الحالي.

### إزالة الغموض بين الدوال ذات الأسماء المتطابقة (Disambiguating Between Identically Named Methods)

لا يوجد شيء في Rust يمنع `trait` من امتلاك دالة بنفس اسم دالة `trait` آخر، ولا تمنعك Rust من تنفيذ كلا السمتين على نوع واحد. من الممكن أيضاً تنفيذ دالة مباشرة على النوع بنفس اسم الدوال من `traits`.

عند استدعاء دوال بنفس الاسم، ستحتاج إلى إخبار Rust بأيها تريد استخدامه. ضع في اعتبارك الكود في القائمة 20-17 حيث عرفنا سمتين، `Pilot` و `Wizard` ، كلاهما يمتلك دالة تسمى `fly`. ثم ننفذ كلا السمتين على نوع `Human` الذي يمتلك بالفعل دالة تسمى `fly` منفذة عليه. كل دالة `fly` تفعل شيئاً مختلفاً.

<Listing number="20-17" file-name="src/main.rs" caption="تعريف سمتين تمتلكان دالة `fly` وتنفيذهما على نوع `Human` ، مع تنفيذ دالة `fly` على `Human` مباشرة">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-17/src/main.rs:here}}
```

</Listing>

عندما نستدعي `fly` على مثيل من `Human` ، يقوم (المترجم) `compiler` افتراضياً باستدعاء الدالة المنفذة مباشرة على النوع، كما هو موضح في القائمة 20-18.

<Listing number="20-18" file-name="src/main.rs" caption="استدعاء `fly` على مثيل من `Human` ">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-18/src/main.rs:here}}
```

</Listing>

سيؤدي تشغيل هذا الكود إلى طباعة `*waving arms furiously*` ، مما يظهر أن Rust استدعت دالة `fly` المنفذة على `Human` مباشرة.

لاستدعاء دوال `fly` من سمة `Pilot` أو سمة `Wizard` ، نحتاج إلى استخدام صيغة أكثر صراحة لتحديد أي دالة `fly` نعنيها. توضح القائمة 20-19 هذه الصيغة.

<Listing number="20-19" file-name="src/main.rs" caption="تحديد دالة `fly` لأي سمة نريد استدعاءها">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-19/src/main.rs:here}}
```

</Listing>

تحديد اسم `trait` قبل اسم الدالة يوضح لـ Rust أي تنفيذ لـ `fly` نريد استدعاءه. يمكننا أيضاً الكتابة (لإزالة الغموض) `disambiguate`.

يؤدي تشغيل هذا الكود إلى طباعة ما يلي:

```console
{{#include ../listings/ch20-advanced-features/listing-20-19/output.txt}}
```

لأن دالة `fly` تأخذ معلمة `self` ، إذا كان لدينا نوعان كلاهما ينفذ `trait` واحداً، فيمكن لـ Rust معرفة أي تنفيذ لـ `trait` يجب استخدامه بناءً على نوع `self`.

ومع ذلك، فإن (الدوال المرتبطة) `associated functions` التي ليست `methods` لا تحتوي على معلمة `self`. عندما يكون هناك عدة أنواع أو `traits` تعرف دوالاً ليست `methods` بنفس اسم الدالة، لا تعرف Rust دائماً النوع الذي تقصده ما لم تستخدم (الصيغة المؤهلة بالكامل) `fully qualified syntax`. على سبيل المثال، في القائمة 20-20، ننشئ `trait` لملجأ حيوانات يريد تسمية جميع الجراء Spot. نصنع سمة `Animal` مع دالة مرتبطة ليست `method` تسمى `baby_name`. يتم تنفيذ سمة `Animal` للهيكل `Dog` ، والذي نوفر عليه أيضاً دالة مرتبطة ليست `method` تسمى `baby_name` مباشرة.

<Listing number="20-20" file-name="src/main.rs" caption="سمة مع دالة مرتبطة ونوع مع دالة مرتبطة بنفس الاسم ينفذ السمة أيضاً">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-20/src/main.rs}}
```

</Listing>

ننفذ الكود لتسمية جميع الجراء Spot في دالة `baby_name` المرتبطة المعرفة على `Dog` مباشرة. ينفذ نوع `Dog` أيضاً سمة `Animal` ، التي تصف الخصائص التي تمتلكها جميع الحيوانات. تسمى صغار الكلاب جراء (puppies)، ويتم التعبير عن ذلك في تنفيذ سمة `Animal` على `Dog` في دالة `baby_name` المرتبطة بسمة `Animal`.

في `main` ، نستدعي دالة `Dog::baby_name` ، التي تستدعي الدالة المرتبطة المعرفة على `Dog` مباشرة. يطبع هذا الكود ما يلي:

```console
{{#include ../listings/ch20-advanced-features/listing-20-20/output.txt}}
```

هذا المخرج ليس ما أردناه. نريد استدعاء دالة `baby_name` التي هي جزء من سمة `Animal` التي نفذناها على `Dog` بحيث يطبع الكود `A baby dog is called a puppy`. تقنية تحديد اسم `trait` التي استخدمناها في القائمة 20-19 لا تساعد هنا؛ إذا قمنا بتغيير `main` إلى الكود في القائمة 20-21، فسنحصل على خطأ في الترجمة.

<Listing number="20-21" file-name="src/main.rs" caption="محاولة استدعاء دالة `baby_name` من سمة `Animal` ، لكن Rust لا تعرف أي تنفيذ تستخدم">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-21/src/main.rs:here}}
```

</Listing>

لأن `Animal::baby_name` لا تحتوي على معلمة `self` ، وقد تكون هناك أنواع أخرى تنفذ سمة `Animal` ، لا تستطيع Rust معرفة أي تنفيذ لـ `Animal::baby_name` نريد. سنحصل على خطأ المترجم هذا:

```console
{{#include ../listings/ch20-advanced-features/listing-20-21/output.txt}}
```

لإزالة الغموض وإخبار Rust أننا نريد استخدام تنفيذ `Animal` لـ `Dog` بدلاً من تنفيذ `Animal` لنوع آخر، نحتاج إلى استخدام `fully qualified syntax`. توضح القائمة 20-22 كيفية استخدام `fully qualified syntax`.

<Listing number="20-22" file-name="src/main.rs" caption="استخدام الصيغة المؤهلة بالكامل لتحديد أننا نريد استدعاء دالة `baby_name` من سمة `Animal` كما هي منفذة على `Dog` ">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-22/src/main.rs:here}}
```

</Listing>

نحن نزود Rust بـ `type annotation` داخل الأقواس الزاوية، مما يشير إلى أننا نريد استدعاء دالة `baby_name` من سمة `Animal` كما هي منفذة على `Dog` من خلال القول بأننا نريد معاملة نوع `Dog` كـ `Animal` لاستدعاء هذه الدالة. سيطبع هذا الكود الآن ما نريد:

```console
{{#include ../listings/ch20-advanced-features/listing-20-22/output.txt}}
```

بشكل عام، يتم تعريف `fully qualified syntax` كما يلي:

```rust,ignore
<Type as Trait>::function(receiver_if_method, next_arg, ...);
```

بالنسبة للدوال المرتبطة التي ليست `methods` ، لن يكون هناك (مستقبل) `receiver`: سيكون هناك فقط قائمة الوسائط الأخرى. يمكنك استخدام `fully qualified syntax` في كل مكان تستدعي فيه دوالاً أو `methods`. ومع ذلك، يُسمح لك بحذف أي جزء من هذه الصيغة يمكن لـ Rust استنتاجه من معلومات أخرى في البرنامج. تحتاج فقط إلى استخدام هذه الصيغة الأكثر تفصيلاً في الحالات التي توجد فيها تطبيقات متعددة تستخدم نفس الاسم وتحتاج Rust إلى مساعدة لتحديد التطبيق الذي تريد استدعاءه.

### استخدام السمات الفائقة (Using Supertraits)

أحياناً قد تكتب تعريف `trait` يعتمد على `trait` آخر: لكي ينفذ نوع ما السمة الأولى، تريد أن تطلب من ذلك النوع أيضاً تنفيذ السمة الثانية. ستفعل ذلك حتى يتمكن تعريف `trait` الخاص بك من الاستفادة من (العناصر المرتبطة) `associated items` للسمة الثانية. يسمى `trait` الذي يعتمد عليه تعريف `trait` الخاص بك بـ (السمة الفائقة) `supertrait` لسمتك.

على سبيل المثال، لنقل إننا نريد صنع سمة `OutlinePrint` مع دالة `outline_print` التي ستطبع قيمة معينة منسقة بحيث تكون مؤطرة بالنجوم. أي، بالنظر إلى هيكل `Point` الذي ينفذ سمة المكتبة القياسية `Display` لتكون النتيجة `(x, y)` ، عندما نستدعي `outline_print` على مثيل `Point` يحتوي على `1` لـ `x` و `3` لـ `y` ، يجب أن يطبع ما يلي:

```text
**********
*        *
* (1, 3) *
*        *
**********
```

في تنفيذ دالة `outline_print` ، نريد استخدام وظائف سمة `Display`. لذلك، نحتاج إلى تحديد أن سمة `OutlinePrint` ستعمل فقط للأنواع التي تنفذ أيضاً `Display` وتوفر الوظائف التي تحتاجها `OutlinePrint`. يمكننا القيام بذلك في تعريف `trait` من خلال تحديد `OutlinePrint: Display`. هذه التقنية تشبه إضافة `trait bound` إلى `trait`. تعرض القائمة 20-23 تنفيذاً لسمة `OutlinePrint`.

<Listing number="20-23" file-name="src/main.rs" caption="تنفيذ سمة `OutlinePrint` التي تتطلب الوظائف من `Display` ">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-23/src/main.rs:here}}
```

</Listing>

لأننا حددنا أن `OutlinePrint` تتطلب سمة `Display` ، يمكننا استخدام دالة `to_string` التي يتم تنفيذها تلقائياً لأي نوع ينفذ `Display`. إذا حاولنا استخدام `to_string` دون إضافة نقطتين وتحديد سمة `Display` بعد اسم `trait` ، فسنحصل على خطأ يقول إنه لم يتم العثور على دالة باسم `to_string` للنوع `&Self` في النطاق الحالي.

دعنا نرى ما يحدث عندما نحاول تنفيذ `OutlinePrint` على نوع لا ينفذ `Display` ، مثل هيكل `Point`:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/src/main.rs:here}}
```

</Listing>

نحصل على خطأ يقول إن `Display` مطلوب ولكنه غير منفذ:

```console
{{#include ../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/output.txt}}
```

لإصلاح ذلك، ننفذ `Display` على `Point` ونلبي القيد الذي تتطلبه `OutlinePrint` ، هكذا:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-03-impl-display-for-point/src/main.rs:here}}
```

</Listing>

بعد ذلك، سيتم تجميع تنفيذ سمة `OutlinePrint` على `Point` بنجاح، ويمكننا استدعاء `outline_print` على مثيل `Point` لعرضه داخل إطار من النجوم.

### تنفيذ السمات الخارجية باستخدام نمط النوع الجديد (Implementing External Traits with the Newtype Pattern)

في قسم ["تنفيذ سمة على نوع"][implementing-a-trait-on-a-type] في الفصل العاشر، ذكرنا (قاعدة اليتيم) `orphan rule` التي تنص على أنه لا يُسمح لنا بتنفيذ `trait` على نوع إلا إذا كان `trait` أو النوع، أو كلاهما، محليين لـ (صندوقنا) `crate`. من الممكن التغلب على هذا القيد باستخدام `newtype pattern` ، والذي يتضمن إنشاء نوع جديد في (هيكل مجموعة) `tuple struct`. (غطينا `tuple structs` في قسم ["إنشاء أنواع مختلفة باستخدام هياكل المجموعات"][tuple-structs] في الفصل الخامس). سيمتلك `tuple struct` حقلاً واحداً ويكون تغليفاً رقيقاً حول النوع الذي نريد تنفيذ `trait` له. بعد ذلك، يكون نوع التغليف محلياً لـ `crate` الخاص بنا، ويمكننا تنفيذ `trait` على التغليف. `Newtype` هو مصطلح ينبع من لغة البرمجة Haskell. لا توجد عقوبة على أداء وقت التشغيل لاستخدام هذا النمط، ويتم حذف نوع التغليف في وقت الترجمة.

كمثال، لنقل إننا نريد تنفيذ `Display` على `Vec<T>` ، وهو ما تمنعنا `orphan rule` من القيام به مباشرة لأن سمة `Display` ونوع `Vec<T>` معرفان خارج `crate` الخاص بنا. يمكننا صنع هيكل `Wrapper` يحمل مثيلاً من `Vec<T>` ؛ ثم يمكننا تنفيذ `Display` على `Wrapper` واستخدام قيمة `Vec<T>` ، كما هو موضح في القائمة 20-24.

<Listing number="20-24" file-name="src/main.rs" caption="إنشاء نوع `Wrapper` حول `Vec<String>` لتنفيذ `Display` ">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-24/src/main.rs}}
```

</Listing>

يستخدم تنفيذ `Display` القيمة `self.0` للوصول إلى `Vec<T>` الداخلي لأن `Wrapper` هو `tuple struct` و `Vec<T>` هو العنصر في الفهرس 0 في المجموعة. بعد ذلك، يمكننا استخدام وظائف سمة `Display` على `Wrapper`.

الجانب السلبي لاستخدام هذه التقنية هو أن `Wrapper` هو نوع جديد، لذا فهو لا يمتلك دوال القيمة التي يحملها. سيتعين علينا تنفيذ جميع دوال `Vec<T>` مباشرة على `Wrapper` بحيث تقوم الدوال بـ (التفويض) `delegate` إلى `self.0` ، مما سيسمح لنا بمعاملة `Wrapper` تماماً مثل `Vec<T>`. إذا أردنا أن يمتلك النوع الجديد كل دالة يمتلكها النوع الداخلي، فإن تنفيذ سمة `Deref` على `Wrapper` لإعادة النوع الداخلي سيكون حلاً (ناقشنا تنفيذ سمة `Deref` في قسم ["معاملة المؤشرات الذكية مثل المراجع العادية"][smart-pointer-deref] في الفصل الخامس عشر). إذا لم نكن نريد أن يمتلك نوع `Wrapper` جميع دوال النوع الداخلي - على سبيل المثال، لتقييد سلوك نوع `Wrapper` - سيتعين علينا تنفيذ الدوال التي نريدها فقط يدوياً.

يعد `newtype pattern` مفيداً أيضاً حتى عندما لا تكون `traits` متضمنة. دعنا نغير التركيز وننظر في بعض الطرق المتقدمة للتفاعل مع نظام أنواع Rust.

[newtype]: ch20-02-advanced-traits.html#implementing-external-traits-with-the-newtype-pattern
[implementing-a-trait-on-a-type]: ch10-02-traits.html#implementing-a-trait-on-a-type
[traits]: ch10-02-traits.html
[smart-pointer-deref]: ch15-02-deref.html#treating-smart-pointers-like-regular-references
[tuple-structs]: ch05-01-defining-structs.html#creating-different-types-with-tuple-structs
