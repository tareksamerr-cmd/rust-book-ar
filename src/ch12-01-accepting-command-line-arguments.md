## استقبال وسائط سطر الأوامر (Command Line Arguments)

لنقم بإنشاء مشروع جديد كالعادة باستخدام `cargo new`. سنطلق على مشروعنا اسم `minigrep` لتمييزه عن أداة `grep` التي قد تكون موجودة بالفعل على نظامك:

```console
$ cargo new minigrep
     Created binary (application) `minigrep` project
$ cd minigrep
```

المهمة الأولى هي جعل `minigrep` يقبل وسيطي (Arguments) سطر الأوامر الخاصين به: مسار الملف وسلسلة نصية (String) للبحث عنها. أي أننا نريد أن نكون قادرين على تشغيل برنامجنا باستخدام `cargo run` متبوعاً بشرطتين للإشارة إلى أن الوسائط (Arguments) التالية مخصصة لبرنامجنا وليس لـ `cargo` نفسه، ثم السلسلة النصية المراد البحث عنها، ومسار الملف المراد البحث فيه، على النحو التالي:

```console
$ cargo run -- searchstring example-filename.txt
```

في الوقت الحالي، لا يمكن للبرنامج الذي تم إنشاؤه بواسطة `cargo new` معالجة Arguments التي نمررها له. يمكن لبعض المكتبات (Libraries) الموجودة على [crates.io](https://crates.io/) المساعدة في كتابة برنامج يقبل Arguments سطر الأوامر، ولكن بما أنك تتعلم هذا المفهوم للتو، فلنقم بتنفيذ هذه الإمكانية بأنفسنا.

### قراءة قيم الوسائط (Argument Values)

لتمكين `minigrep` من قراءة قيم Arguments سطر الأوامر التي نمررها إليه، سنحتاج إلى الدالة (Function) ‏`std::env::args` المتوفرة في مكتبة Rust القياسية (Standard Library). تعيد هذه Function مكرراً (Iterator) لوسائط سطر الأوامر التي تم تمريرها إلى `minigrep`. سنغطي Iterators بالكامل في [الفصل 13][ch13]<!-- ignore -->. في الوقت الحالي، تحتاج فقط إلى معرفة تفصيلين حول Iterators: تنتج Iterators سلسلة من القيم، ويمكننا استدعاء التابع (Method) ‏`collect` على Iterator لتحويله إلى مجموعة (Collection)، مثل المتجه (Vector)، الذي يحتوي على جميع العناصر التي ينتجها Iterator.

تسمح الشفرة البرمجية (Code) في القائمة 12-1 لبرنامج `minigrep` الخاص بك بقراءة أي Arguments سطر أوامر يتم تمريرها إليه ثم جمع القيم في Vector.

<Listing number="12-1" file-name="src/main.rs" caption="جمع وسائط سطر الأوامر في متجه وطباعتها">

```rust
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-01/src/main.rs}}
```

</Listing>

أولاً، نقوم بجلب الوحدة (Module) ‏`std::env` إلى النطاق (Scope) باستخدام عبارة (Statement) ‏`use` حتى نتمكن من استخدام Function ‏`args` الخاصة بها. لاحظ أن Function ‏`std::env::args` متداخلة في مستويين من Modules. كما ناقشنا في [الفصل 7][ch7-idiomatic-use]<!-- ignore -->، في الحالات التي تكون فيها Function المطلوبة متداخلة في أكثر من Module واحد، اخترنا جلب Module الأب إلى Scope بدلاً من Function نفسها. ومن خلال القيام بذلك، يمكننا بسهولة استخدام Functions أخرى من `std::env`. كما أنه أقل غموضاً من إضافة `use std::env::args` ثم استدعاء Function باستخدام `args` فقط، لأن `args` قد يتم الخلط بينها وبين Function معرفة في Module الحالي.

> ### الدالة `args` والترميز الموحد (Unicode) غير الصالح
>
> لاحظ أن `std::env::args` ستتسبب في حالة ذعر (Panic) إذا كان أي Argument يحتوي على Unicode غير صالح. إذا كان برنامجك يحتاج إلى قبول Arguments تحتوي على Unicode غير صالح، فاستخدم `std::env::args_os` بدلاً من ذلك. تعيد تلك Function مكرراً (Iterator) ينتج قيم `OsString` بدلاً من قيم `String`. لقد اخترنا استخدام `std::env::args` هنا للتبسيط لأن قيم `OsString` تختلف باختلاف نظام التشغيل (Platform) وهي أكثر تعقيداً في التعامل معها من قيم `String`.

في السطر الأول من `main` استدعينا `env::args` واستخدمنا `collect` على الفور لتحويل Iterator إلى Vector يحتوي على جميع القيم التي ينتجها Iterator. يمكننا استخدام Function ‏`collect` لإنشاء أنواع عديدة من Collections، لذا قمنا بتحديد نوع (Type) ‏`args` صراحة لنبين أننا نريد Vector من السلاسل النصية (Strings). على الرغم من أنك نادراً ما تحتاج إلى تحديد Types في Rust، إلا أن `collect` هي إحدى Functions التي غالباً ما تحتاج إلى تحديد نوعها لأن Rust لا يستطيع استنتاج نوع Collection الذي تريده.

أخيراً، نقوم بطباعة Vector باستخدام ماكرو التصحيح (Debug Macro). لنحاول تشغيل Code أولاً بدون Arguments ثم مع وسيطين:

```console
{{#include ../listings/ch12-an-io-project/listing-12-01/output.txt}}
```

```console
{{#include ../listings/ch12-an-io-project/output-only-01-with-args/output.txt}}
```

لاحظ أن القيمة الأولى في Vector هي `"target/debug/minigrep"`، وهي اسم ملفنا الثنائي (Binary). هذا يطابق سلوك قائمة Arguments في لغة C، مما يسمح للبرامج باستخدام الاسم الذي تم استدعاؤها به أثناء تنفيذها. غالباً ما يكون من الملائم الوصول إلى اسم البرنامج في حال كنت ترغب في طباعته في الرسائل أو تغيير سلوك البرنامج بناءً على الاسم المستعار (Alias) لسطر الأوامر الذي تم استخدامه لاستدعاء البرنامج. ولكن لأغراض هذا الفصل، سنتجاهله ونحفظ فقط وسيطي (Arguments) اللذين نحتاجهما.

### حفظ قيم الوسائط في متغيرات (Variables)

البرنامج قادر حالياً على الوصول إلى القيم المحددة كـ Arguments سطر الأوامر. نحتاج الآن إلى حفظ قيم وسيطي (Arguments) في متغيرات (Variables) حتى نتمكن من استخدام القيم في بقية البرنامج. نقوم بذلك في القائمة 12-2.

<Listing number="12-2" file-name="src/main.rs" caption="إنشاء متغيرات للاحتفاظ بوسيط الاستعلام ومسار الملف">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-02/src/main.rs}}
```

</Listing>

كما رأينا عندما قمنا بطباعة Vector، فإن اسم البرنامج يشغل القيمة الأولى في Vector عند `args[0]`، لذا سنبدأ Arguments عند الفهرس (Index) 1. أول Argument يأخذه `minigrep` هو السلسلة النصية التي نبحث عنها، لذا نضع مرجعاً (Reference) لـ Argument الأول في المتغير (Variable) ‏`query`. سيكون Argument الثاني هو مسار الملف، لذا نضع Reference لـ Argument الثاني في Variable ‏`file_path`.

نقوم بطباعة قيم هذه Variables مؤقتاً لإثبات أن Code يعمل كما نريد. لنقم بتشغيل هذا البرنامج مرة أخرى مع Arguments ‏`test` و `sample.txt`:

```console
{{#include ../listings/ch12-an-io-project/listing-12-02/output.txt}}
```

رائع، البرنامج يعمل! يتم حفظ قيم Arguments التي نحتاجها في Variables الصحيحة. لاحقاً سنضيف بعض معالجة الأخطاء (Error Handling) للتعامل مع بعض المواقف الخاطئة المحتملة، مثل عندما لا يقدم المستخدم أي Arguments؛ في الوقت الحالي، سنتجاهل هذا الموقف ونعمل على إضافة إمكانيات قراءة الملفات بدلاً من ذلك.

[ch13]: ch13-00-functional-features.html
[ch7-idiomatic-use]: ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#creating-idiomatic-use-paths
