## أنواع البيانات العامة (Generic Data Types)

نستخدم الـ generics لإنشاء تعريفات لعناصر مثل توقيعات الدوال (function signatures) أو الـ structs، والتي يمكننا بعد ذلك استخدامها مع العديد من أنواع البيانات الملموسة (concrete data types) المختلفة. دعنا ننظر أولاً إلى كيفية تعريف الـ functions، الـ structs، الـ enums، والـ methods باستخدام الـ generics. بعد ذلك، سنناقش كيف تؤثر الـ generics على أداء الكود.

### في تعريفات الدوال

عند تعريف function تستخدم الـ generics، نضع الـ generics في توقيع الـ function حيث نحدد عادةً أنواع البيانات (data types) للمعلمات (parameters) وقيمة الإرجاع (return value). يؤدي القيام بذلك إلى جعل الكود الخاص بنا أكثر مرونة ويوفر وظائف أكثر لمستدعي الـ function مع منع تكرار الكود.

بالاستمرار مع الـ function `largest`، تُظهر القائمة 10-4 دالتين تجدان أكبر قيمة في شريحة (slice). سنقوم بعد ذلك بدمج هاتين الدالتين في function واحدة تستخدم الـ generics.

<Listing number="10-4" file-name="src/main.rs" caption="دالتان تختلفان فقط في أسمائهما وفي الأنواع في توقيعاتهما">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-04/src/main.rs:here}}
```

</Listing>

الدالة `largest_i32` هي الدالة التي استخرجناها في القائمة 10-3 والتي تجد أكبر `i32` في slice. تجد الدالة `largest_char` أكبر `char` في slice. تحتوي نصوص الدوال على نفس الكود، لذا دعنا نتخلص من التكرار عن طريق إدخال معلمة نوع عام (generic type parameter) في function واحدة.

لتعيين الـ types كـ parameters في function واحدة جديدة، نحتاج إلى تسمية الـ type parameter، تمامًا كما نفعل لـ value parameters لـ function. يمكنك استخدام أي معرف (identifier) كاسم لـ type parameter. لكننا سنستخدم `T` لأنه، حسب الاصطلاح، تكون أسماء الـ type parameter في Rust قصيرة، وغالبًا ما تكون حرفًا واحدًا فقط، واصطلاح تسمية الـ type في Rust هو UpperCamelCase. `T`، اختصار لـ *type*، هو الخيار الافتراضي لمعظم مبرمجي Rust.

عندما نستخدم parameter في نص الـ function، يجب علينا الإعلان عن اسم الـ parameter في التوقيع حتى يعرف المترجم (compiler) ما يعنيه هذا الاسم. وبالمثل، عندما نستخدم اسم type parameter في توقيع function، يجب علينا الإعلان عن اسم الـ type parameter قبل استخدامه. لتعريف الـ generic function `largest`، نضع إعلانات اسم الـ type داخل أقواس زاوية، `<>`، بين اسم الـ function وقائمة الـ parameters، مثل هذا:

```rust,ignore
fn largest<T>(list: &[T]) -> &T {
```

نقرأ هذا التعريف على أنه "الدالة `largest` عامة (generic) على نوع ما `T`." تحتوي هذه الـ function على parameter واحد يسمى `list`، وهو slice من القيم من النوع `T`. ستُرجع الدالة `largest` مرجعًا (reference) إلى قيمة من نفس النوع `T`.

تُظهر القائمة 10-5 تعريف الـ function `largest` المدمج باستخدام نوع البيانات العام (generic data type) في توقيعها. تُظهر القائمة أيضًا كيف يمكننا استدعاء الـ function إما بـ slice من قيم `i32` أو قيم `char`. لاحظ أن هذا الكود لن يتم تجميعه بعد.

<Listing number="10-5" file-name="src/main.rs" caption="الدالة `largest` تستخدم generic type parameters؛ هذا لا يتم تجميعه بعد">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-05/src/main.rs}}
```

</Listing>

إذا قمنا بـ compile هذا الكود الآن، فسنحصل على هذا الخطأ:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-05/output.txt}}
```

يذكر نص المساعدة `std::cmp::PartialOrd`، وهو سمة (trait)، وسنتحدث عن الـ traits في القسم التالي. في الوقت الحالي، اعلم أن هذا الخطأ ينص على أن نص `largest` لن يعمل لجميع الـ types الممكنة التي يمكن أن يكونها `T`. نظرًا لأننا نريد مقارنة قيم من النوع `T` في النص، يمكننا فقط استخدام الـ types التي يمكن ترتيب قيمها. لتمكين المقارنات، تحتوي المكتبة القياسية (standard library) على الـ trait `std::cmp::PartialOrd` الذي يمكنك تطبيقه على الـ types (راجع الملحق ج لمزيد من المعلومات حول هذا الـ trait). لإصلاح القائمة 10-5، يمكننا اتباع اقتراح نص المساعدة وتقييد الـ types الصالحة لـ `T` على تلك التي تطبق `PartialOrd` فقط. سيتم بعد ذلك تجميع القائمة، لأن الـ standard library تطبق `PartialOrd` على كل من `i32` و `char`.

### في تعريفات الـ Struct

يمكننا أيضًا تعريف الـ structs لاستخدام generic type parameter في حقل واحد أو أكثر باستخدام بناء جملة `<>`. تحدد القائمة 10-6 struct `Point<T>` للاحتفاظ بقيم إحداثيات `x` و `y` من أي نوع.

<Listing number="10-6" file-name="src/main.rs" caption="struct `Point<T>` يحتفظ بقيم `x` و `y` من النوع `T`">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-06/src/main.rs}}
```

</Listing>

بناء الجملة لاستخدام الـ generics في تعريفات الـ struct مشابه لذلك المستخدم في تعريفات الـ function. أولاً، نعلن عن اسم الـ type parameter داخل أقواس زاوية مباشرة بعد اسم الـ struct. بعد ذلك، نستخدم الـ generic type في تعريف الـ struct حيث كنا سنحدد أنواع البيانات الملموسة.

لاحظ أنه نظرًا لأننا استخدمنا generic type واحدًا فقط لتعريف `Point<T>`، فإن هذا التعريف يقول إن struct `Point<T>` عام على نوع ما `T`، وأن الـ fields `x` و `y` هما *كلاهما* من نفس النوع، مهما كان هذا النوع. إذا أنشأنا مثيلًا لـ `Point<T>` يحتوي على قيم من أنواع مختلفة، كما في القائمة 10-7، فلن يتم تجميع الكود الخاص بنا.

<Listing number="10-7" file-name="src/main.rs" caption="يجب أن يكون الـ fields `x` و `y` من نفس النوع لأن كلاهما له نفس generic data type `T`.">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-07/src/main.rs}}
```

</Listing>

في هذا المثال، عندما نخصص القيمة الصحيحة `5` لـ `x`، فإننا نُعلم الـ compiler أن الـ generic type `T` سيكون عددًا صحيحًا لهذا المثيل من `Point<T>`. بعد ذلك، عندما نحدد `4.0` لـ `y`، والذي حددناه ليكون من نفس نوع `x`، سنحصل على خطأ عدم تطابق النوع (type mismatch) مثل هذا:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-07/output.txt}}
```

لتعريف struct `Point` حيث يكون `x` و `y` كلاهما generics ولكنهما يمكن أن يكونا من أنواع مختلفة، يمكننا استخدام multiple generic type parameters. على سبيل المثال، في القائمة 10-8، نغير تعريف `Point` ليكون عامًا على الـ types `T` و `U` حيث يكون `x` من النوع `T` و `y` من النوع `U`.

<Listing number="10-8" file-name="src/main.rs" caption="struct `Point<T, U>` عام على نوعين بحيث يمكن أن يكون `x` و `y` قيمًا من أنواع مختلفة">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-08/src/main.rs}}
```

</Listing>

الآن جميع مثيلات `Point` الموضحة مسموح بها! يمكنك استخدام أي عدد تريده من generic type parameters في تعريف، ولكن استخدام أكثر من عدد قليل يجعل الكود الخاص بك صعب القراءة. إذا وجدت أنك بحاجة إلى الكثير من الـ generic types في الكود الخاص بك، فقد يشير ذلك إلى أن الكود الخاص بك يحتاج إلى إعادة هيكلة (restructuring) إلى أجزاء أصغر.

### في تعريفات الـ Enum

كما فعلنا مع الـ structs، يمكننا تعريف الـ enums للاحتفاظ بـ generic data types في متغيراتها (variants). دعنا نلقي نظرة أخرى على الـ enum `Option<T>` الذي توفره الـ standard library، والذي استخدمناه في الفصل 6:

```rust
enum Option<T> {
    Some(T),
    None,
}
```

يجب أن يكون هذا التعريف أكثر منطقية بالنسبة لك الآن. كما ترى، فإن الـ enum `Option<T>` عام على النوع `T` وله متغيران: `Some`، الذي يحتوي على قيمة واحدة من النوع `T`، ومتغير `None` الذي لا يحتوي على أي قيمة. باستخدام الـ enum `Option<T>`، يمكننا التعبير عن المفهوم المجرد للقيمة الاختيارية (optional value)، ولأن `Option<T>` عام، يمكننا استخدام هذا التجريد بغض النظر عن نوع القيمة الاختيارية.

يمكن أن تستخدم الـ Enums أنواعًا عامة متعددة أيضًا. تعريف الـ enum `Result` الذي استخدمناه في الفصل 9 هو أحد الأمثلة:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

الـ enum `Result` عام على نوعين، `T` و `E`، وله متغيران: `Ok`، الذي يحتوي على قيمة من النوع `T`، و `Err`، الذي يحتوي على قيمة من النوع `E`. يجعل هذا التعريف من الملائم استخدام الـ enum `Result` في أي مكان لدينا عملية قد تنجح (تُرجع قيمة من نوع ما `T`) أو تفشل (تُرجع خطأ من نوع ما `E`). في الواقع، هذا ما استخدمناه لفتح ملف في القائمة 9-3، حيث تم ملء `T` بالنوع `std::fs::File` عندما تم فتح الملف بنجاح وتم ملء `E` بالنوع `std::io::Error` عندما كانت هناك مشاكل في فتح الملف.

عندما تدرك مواقف في الكود الخاص بك مع تعريفات struct أو enum متعددة تختلف فقط في أنواع القيم التي تحتفظ بها، يمكنك تجنب التكرار باستخدام generic types بدلاً من ذلك.

### في تعريفات الـ Method

يمكننا تطبيق الـ methods على الـ structs والـ enums (كما فعلنا في الفصل 5) واستخدام generic types في تعريفاتها أيضًا. تُظهر القائمة 10-9 struct `Point<T>` الذي عرفناه في القائمة 10-6 مع method يسمى `x` مطبق عليه.

<Listing number="10-9" file-name="src/main.rs" caption="تطبيق method يسمى `x` على struct `Point<T>` الذي سيُرجع مرجعًا إلى الـ field `x` من النوع `T`">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-09/src/main.rs}}
```

</Listing>

هنا، قمنا بتعريف method يسمى `x` على `Point<T>` يُرجع reference إلى البيانات الموجودة في الـ field `x`.

لاحظ أنه يجب علينا الإعلان عن `T` مباشرة بعد `impl` حتى نتمكن من استخدام `T` لتحديد أننا نطبق الـ methods على النوع `Point<T>`. من خلال الإعلان عن `T` كـ generic type بعد `impl`، يمكن لـ Rust تحديد أن النوع الموجود في الأقواس الزاوية في `Point` هو generic type بدلاً من concrete type. كان بإمكاننا اختيار اسم مختلف لـ generic parameter هذا عن الـ generic parameter المعلن في تعريف الـ struct، ولكن استخدام نفس الاسم هو اصطلاح. إذا كتبت method داخل `impl` يعلن عن generic type، فسيتم تعريف هذا الـ method على أي مثيل من النوع، بغض النظر عن الـ concrete type الذي ينتهي به الأمر ليحل محل الـ generic type.

يمكننا أيضًا تحديد قيود (constraints) على الـ generic types عند تعريف الـ methods على النوع. يمكننا، على سبيل المثال، تطبيق الـ methods فقط على مثيلات `Point<f32>` بدلاً من مثيلات `Point<T>` بأي generic type. في القائمة 10-10، نستخدم الـ concrete type `f32`، مما يعني أننا لا نعلن عن أي types بعد `impl`.

<Listing number="10-10" file-name="src/main.rs" caption="كتلة `impl` تنطبق فقط على struct بنوع ملموس معين لـ generic type parameter `T`">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-10/src/main.rs:here}}
```

</Listing>

يعني هذا الكود أن النوع `Point<f32>` سيكون له method `distance_from_origin`؛ لن يكون لدى المثيلات الأخرى من `Point<T>` حيث `T` ليس من النوع `f32` هذا الـ method المعرف. يقيس الـ method مدى بعد نقطتنا عن النقطة عند الإحداثيات (0.0، 0.0) ويستخدم العمليات الرياضية المتاحة فقط لـ floating-point types.

لا تكون generic type parameters في تعريف struct دائمًا هي نفسها التي تستخدمها في توقيعات الـ method لنفس الـ struct. تستخدم القائمة 10-11 الـ generic types `X1` و `Y1` لـ struct `Point` و `X2` و `Y2` لـ method signature `mixup` لجعل المثال أكثر وضوحًا. ينشئ الـ method مثيل `Point` جديدًا بقيمة `x` من `self` `Point` (من النوع `X1`) وقيمة `y` من `Point` الذي تم تمريره (من النوع `Y2`).

<Listing number="10-11" file-name="src/main.rs" caption="method يستخدم generic types تختلف عن تعريف struct الخاص به">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-11/src/main.rs}}
```

</Listing>

في `main`، قمنا بتعريف `Point` يحتوي على `i32` لـ `x` (بقيمة `5`) و `f64` لـ `y` (بقيمة `10.4`). المتغير `p2` هو struct `Point` يحتوي على string slice لـ `x` (بقيمة `"Hello"`) و `char` لـ `y` (بقيمة `c`). استدعاء `mixup` على `p1` بالوسيطة `p2` يعطينا `p3`، وستطبع نتيجة الاستدعاء `p3.x = 5, p3.y = c`.

الغرض من هذا المثال هو إظهار موقف يتم فيه الإعلان عن بعض الـ generic parameters باستخدام `impl` ويتم الإعلان عن البعض الآخر باستخدام تعريف الـ method. هنا، يتم الإعلان عن الـ generic parameters `X1` و `Y1` بعد `impl` لأنهما يذهبان مع تعريف الـ struct. يتم الإعلان عن الـ generic parameters `X2` و `Y2` بعد `fn mixup` لأنهما لا يتعلقان إلا بالـ method.

### أداء الكود الذي يستخدم الـ Generics

قد تتساءل عما إذا كانت هناك تكلفة في وقت التشغيل (runtime cost) عند استخدام generic type parameters. الخبر السار هو أن استخدام generic types لن يجعل برنامجك يعمل أبطأ مما لو كان بأنواع ملموسة (concrete types).

تحقق Rust ذلك عن طريق إجراء *تنميط أحادي* (monomorphization) للكود باستخدام الـ generics في الـ compile time. الـ *Monomorphization* هي عملية تحويل الـ generic code إلى كود محدد عن طريق ملء الـ concrete types التي يتم استخدامها عند الـ compile. في هذه العملية، يقوم الـ compiler بعكس الخطوات التي استخدمناها لإنشاء الـ generic function في القائمة 10-5: ينظر الـ compiler إلى جميع الأماكن التي يتم فيها استدعاء الـ generic code ويولد كودًا لـ concrete types التي يتم استدعاء الـ generic code بها.

دعنا ننظر إلى كيفية عمل ذلك باستخدام الـ generic enum `Option<T>` الخاص بالـ standard library:

```rust
let integer = Some(5);
let float = Some(5.0);
```

عندما تقوم Rust بـ compile هذا الكود، فإنها تجري monomorphization. خلال هذه العملية، يقرأ الـ compiler القيم التي تم استخدامها في مثيلات `Option<T>` ويحدد نوعين من `Option<T>`: أحدهما `i32` والآخر `f64`. على هذا النحو، فإنه يوسع التعريف العام لـ `Option<T>` إلى تعريفين متخصصين لـ `i32` و `f64`، وبالتالي يحل محل التعريف العام بالتعريفات المحددة.

يبدو الإصدار الذي تم عمل monomorphization له من الكود مشابهًا لما يلي (يستخدم الـ compiler أسماء مختلفة عما نستخدمه هنا للتوضيح):

<Listing file-name="src/main.rs">

```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

</Listing>

يتم استبدال الـ generic `Option<T>` بالتعريفات المحددة التي أنشأها الـ compiler. نظرًا لأن Rust تقوم بـ compile الـ generic code إلى كود يحدد النوع في كل مثيل، فإننا لا ندفع أي runtime cost لاستخدام الـ generics. عندما يتم تشغيل الكود، فإنه يعمل تمامًا كما لو كنا قد كررنا كل تعريف يدويًا. عملية الـ monomorphization تجعل الـ generics في Rust فعالة للغاية في الـ runtime.
