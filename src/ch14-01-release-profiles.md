## تخصيص عمليات البناء باستخدام ملفات تعريف الإصدار (Release Profiles)

في لغة Rust، تعتبر *ملفات تعريف الإصدار* (Release Profiles) ملفات تعريف محددة مسبقاً وقابلة للتخصيص مع إعدادات (Configurations) مختلفة تسمح للمبرمج بمزيد من التحكم في الخيارات المتنوعة لترجمة الشفرة البرمجية (Compiling Code). يتم تكوين كل ملف تعريف (Profile) بشكل مستقل عن الملفات الأخرى.

تمتلك أداة Cargo ملفي تعريف رئيسيين: ملف التعريف `dev` الذي تستخدمه Cargo عند تشغيل `cargo build` وملف التعريف `release` الذي تستخدمه Cargo عند تشغيل `cargo build --release`. يتم تعريف Profile ‏`dev` بإعدادات افتراضية جيدة للتطوير (Development)، بينما يمتلك Profile ‏`release` إعدادات افتراضية جيدة لإصدارات البناء (Release Builds).

قد تكون أسماء Profiles هذه مألوفة لك من مخرجات عمليات البناء (Builds) الخاصة بك:

<!-- manual-regeneration
anywhere, run:
cargo build
cargo build --release
and ensure output below is accurate
-->

```console
$ cargo build
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.00s
$ cargo build --release
    Finished `release` profile [optimized] target(s) in 0.32s
```

إن `dev` و `release` هما ملفات تعريف مختلفة يستخدمها المترجم (Compiler).

تمتلك Cargo إعدادات افتراضية لكل من Profiles التي يتم تطبيقها عندما لا تكون قد أضفت صراحة أي أقسام `[profile.*]` في ملف *Cargo.toml* الخاص بالمشروع. من خلال إضافة أقسام `[profile.*]` لأي Profile تريد تخصيصه، فإنك تقوم بتجاوز (Override) أي مجموعة فرعية من الإعدادات الافتراضية. على سبيل المثال، إليك القيم الافتراضية لإعداد `opt-level` لملفي التعريف `dev` و `release`:

<span class="filename">اسم الملف: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

يتحكم إعداد `opt-level` في عدد التحسينات (Optimizations) التي ستطبقها Rust على Code الخاص بك، بنطاق يتراوح من 0 إلى 3. يؤدي تطبيق المزيد من Optimizations إلى إطالة وقت الترجمة (Compiling Time)، لذا إذا كنت في مرحلة Development وتقوم بترجمة Code الخاص بك غالباً، فستحتاج إلى عدد أقل من Optimizations للترجمة بشكل أسرع حتى لو كان Code الناتج يعمل بشكل أبطأ. لذا فإن `opt-level` الافتراضي لـ `dev` هو `0`. عندما تكون مستعداً لإصدار Code الخاص بك، فمن الأفضل قضاء وقت أطول في الترجمة. ستقوم بالترجمة في وضع الإصدار (Release Mode) مرة واحدة فقط، ولكنك ستشغل البرنامج المترجم عدة مرات، لذا فإن Release Mode يقايض وقت الترجمة الأطول بشفرة برمجية تعمل بشكل أسرع. وهذا هو السبب في أن `opt-level` الافتراضي لـ Profile ‏`release` هو `3`.

يمكنك تجاوز إعداد افتراضي عن طريق إضافة قيمة مختلفة له في *Cargo.toml*. على سبيل المثال، إذا أردنا استخدام مستوى التحسين 1 في Profile التطوير، يمكننا إضافة هذين السطرين إلى ملف *Cargo.toml* الخاص بمشروعنا:

<span class="filename">اسم الملف: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 1
```

يتجاوز هذا Code الإعداد الافتراضي البالغ `0`. الآن عند تشغيل `cargo build` ستستخدم Cargo الإعدادات الافتراضية لـ Profile ‏`dev` بالإضافة إلى تخصيصنا لـ `opt-level`. ولأننا ضبطنا `opt-level` على `1` فستطبق Cargo المزيد من Optimizations أكثر من الافتراضي، ولكن ليس بقدر ما هو موجود في Release Build.

للحصول على القائمة الكاملة لخيارات التكوين والإعدادات الافتراضية لكل Profile، راجع [وثائق Cargo](https://doc.rust-lang.org/cargo/reference/profiles.html).
