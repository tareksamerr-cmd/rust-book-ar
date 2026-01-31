## الحزم والصناديق (Packages and Crates)

الأجزاء الأولى من نظام الوحدات (Module System) التي سنغطيها هي الحزم (Packages) والصناديق (Crates).

الـ "صندوق" (Crate) هو أصغر كمية من الكود يأخذها مترجم Rust (Compiler) في الاعتبار في المرة الواحدة. حتى لو قمت بتشغيل `rustc` بدلاً من `cargo` ومررت ملف كود مصدري واحد (كما فعلنا سابقاً في قسم ["أساسيات برنامج Rust"][basics] في الفصل 1)، فإن الـ Compiler يعتبر ذلك الملف بمثابة Crate. يمكن أن تحتوي الـ Crates على وحدات (Modules)، وقد يتم تعريف الـ Modules في ملفات أخرى يتم تجميعها مع الـ Crate، كما سنرى في الأقسام القادمة.

يمكن أن يأتي الـ Crate في أحد شكلين: "صندوق ثنائي" (Binary Crate) أو "صندوق مكتبة" (Library Crate). الـ "Binary Crates" هي برامج يمكنك تجميعها إلى ملف قابل للتنفيذ يمكنك تشغيله، مثل برنامج سطر أوامر أو خادم. يجب أن يحتوي كل منها على دالة تسمى `main` تحدد ما يحدث عند تشغيل الملف القابل للتنفيذ. جميع الـ Crates التي أنشأناها حتى الآن كانت Binary Crates.

الـ "Library Crates" لا تحتوي على دالة `main` ولا يتم تجميعها إلى ملف قابل للتنفيذ. بدلاً من ذلك، فهي تحدد وظائف مخصصة للمشاركة مع مشاريع متعددة. على سبيل المثال، يوفر Crate الـ `rand` الذي استخدمناه في [الفصل 2][rand] وظائف تولد أرقاماً عشوائية. في معظم الأوقات عندما يقول مبرمجو Rust (Rustaceans) كلمة "Crate"، فإنهم يقصدون Library Crate، ويستخدمون كلمة "Crate" بالتبادل مع المفهوم البرمجي العام لـ "المكتبة" (Library).

"جذر الصندوق" (Crate Root) هو ملف مصدري يبدأ منه الـ Compiler ويشكل الـ Module الجذر للـ Crate الخاصة بك (سنشرح الـ Modules بعمق في قسم ["التحكم في النطاق والخصوصية باستخدام الوحدات"][modules]).

الـ "حزمة" (Package) هي حزمة من Crate واحد أو أكثر توفر مجموعة من الوظائف. تحتوي الـ Package على ملف *Cargo.toml* يصف كيفية بناء تلك الـ Crates. في الواقع، Cargo هو Package يحتوي على Binary Crate لأداة سطر الأوامر التي كنت تستخدمها لبناء الكود الخاص بك. تحتوي Package الـ Cargo أيضاً على Library Crate يعتمد عليه الـ Binary Crate. يمكن للمشاريع الأخرى الاعتماد على Library Crate الخاص بـ Cargo لاستخدام نفس المنطق الذي تستخدمه أداة سطر أوامر Cargo.

يمكن أن تحتوي الـ Package على أي عدد تريده من الـ Binary Crates، ولكن على الأكثر Library Crate واحد فقط. يجب أن تحتوي الـ Package على Crate واحد على الأقل، سواء كان ذلك Library Crate أو Binary Crate.

دعنا نستعرض ما يحدث عندما ننشئ Package. أولاً، ندخل الأمر `cargo new my-project`:

```console
$ cargo new my-project
     Created binary (application) `my-project` package
$ ls my-project
Cargo.toml
src
$ ls my-project/src
main.rs
```

بعد تشغيل `cargo new my-project` نستخدم `ls` لنرى ما ينشئه Cargo. في دليل *my-project* يوجد ملف *Cargo.toml* مما يمنحنا Package. يوجد أيضاً دليل *src* يحتوي على *main.rs*. افتح *Cargo.toml* في محرر النصوص الخاص بك ولاحظ أنه لا يوجد ذكر لـ *src/main.rs*. يتبع Cargo اتفاقاً (Convention) مفاده أن *src/main.rs* هو الـ Crate Root لـ Binary Crate يحمل نفس اسم الـ Package. وبالمثل، يعرف Cargo أنه إذا كان دليل الـ Package يحتوي على *src/lib.rs* فإن الـ Package تحتوي على Library Crate بنفس اسم الـ Package، ويكون *src/lib.rs* هو الـ Crate Root الخاص به. يمرر Cargo ملفات الـ Crate Root إلى `rustc` لبناء المكتبة أو الملف الثنائي.

هنا، لدينا Package تحتوي فقط على *src/main.rs* مما يعني أنها تحتوي فقط على Binary Crate يسمى `my-project`. إذا كانت الـ Package تحتوي على *src/main.rs* و *src/lib.rs* فإنها تحتوي على صندوقين: Binary و Library، وكلاهما بنفس اسم الـ Package. يمكن أن تحتوي الـ Package على عدة Binary Crates عن طريق وضع الملفات في دليل *src/bin*: سيكون كل ملف عبارة عن Binary Crate منفصل.

[basics]: ch01-02-hello-world.html#rust-program-basics
[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html
[rand]: ch02-00-guessing-game-tutorial.html#generating-a-random-number
