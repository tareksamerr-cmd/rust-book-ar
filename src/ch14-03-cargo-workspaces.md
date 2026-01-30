## مساحات عمل Cargo (Cargo Workspaces)

في الفصل الثاني عشر، قمنا ببناء حزمة (package) تضمنت صندوقاً ثنائياً (binary crate) وصندوق مكتبة (library crate). مع تطور مشروعك، قد تجد أن library crate يستمر في الكبر وترغب في تقسيم الـ package بشكل أكبر إلى عدة library crates. يوفر Cargo ميزة تسمى _مساحات العمل_ (workspaces) يمكنها المساعدة في إدارة عدة packages مرتبطة يتم تطويرها جنباً إلى جنب.

### إنشاء مساحة عمل (Creating a Workspace)

_مساحة العمل_ (workspace) هي مجموعة من الـ packages التي تشترك في نفس ملف _Cargo.lock_ ودليل المخرجات (output directory). دعونا ننشئ مشروعاً باستخدام workspace - سنستخدم كوداً بسيطاً حتى نتمكن من التركيز على هيكل الـ workspace. هناك طرق متعددة لهيكلة الـ workspace، لذا سنعرض طريقة واحدة شائعة. سيكون لدينا workspace يحتوي على binary واثنين من libraries. الـ binary، الذي سيوفر الوظيفة الرئيسية، سيعتمد على المكتبتين. ستوفر إحدى المكتبات دالة (function) باسم `add_one` والمكتبة الأخرى function باسم `add_two`. ستكون هذه الـ crates الثلاثة جزءاً من نفس الـ workspace. سنبدأ بإنشاء دليل جديد للـ workspace:

```console
$ mkdir add
$ cd add
```

بعد ذلك، في دليل _add_ ، ننشئ ملف _Cargo.toml_ الذي سيقوم بتهيئة الـ workspace بالكامل. لن يحتوي هذا الملف على قسم `[package]`. بدلاً من ذلك، سيبدأ بقسم `[workspace]` الذي سيسمح لنا بإضافة أعضاء (members) إلى الـ workspace. نحرص أيضاً على استخدام أحدث وأفضل إصدار من خوارزمية الحل (resolver algorithm) الخاصة بـ Cargo في الـ workspace الخاص بنا عن طريق تعيين قيمة `resolver` إلى `"3"`:

<span class="filename">اسم الملف: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-01-workspace/add/Cargo.toml}}
```

بعد ذلك، سننشئ binary crate المسمى `adder` عن طريق تشغيل `cargo new` داخل دليل _add_:

```console
$ cargo new adder
     Created binary (application) `adder` package
      Adding `adder` as member of workspace at `file:///projects/add`
```

يؤدي تشغيل `cargo new` داخل workspace أيضاً إلى إضافة الـ package المنشأ حديثاً تلقائياً إلى مفتاح `members` في تعريف `[workspace]` في ملف _Cargo.toml_ الخاص بالـ workspace، هكذا:

```toml
{{#include ../listings/ch14-more-about-cargo/output-only-01-adder-crate/add/Cargo.toml}}
```

في هذه المرحلة، يمكننا بناء الـ workspace عن طريق تشغيل `cargo build`. يجب أن تبدو الملفات في دليل _add_ الخاص بك هكذا:

```text
├── Cargo.lock
├── Cargo.toml
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

يحتوي الـ workspace على دليل _target_ واحد في المستوى الأعلى حيث سيتم وضع العناصر المنتجة (artifacts) المجمعة؛ لا يمتلك package الـ `adder` دليل _target_ خاصاً به. حتى لو قمنا بتشغيل `cargo build` من داخل دليل _adder_ ، فإن الـ artifacts المجمعة ستنتهي في _add/target_ بدلاً من _add/adder/target_. يقوم Cargo بهيكلة دليل _target_ في workspace هكذا لأن الـ crates في workspace من المفترض أن تعتمد على بعضها البعض. إذا كان لكل crate دليل _target_ خاص به، فسيضطر كل crate إلى إعادة تجميع كل من الـ crates الأخرى في الـ workspace لوضع الـ artifacts في دليل _target_ الخاص به. من خلال مشاركة دليل _target_ واحد، يمكن للـ crates تجنب إعادة البناء غير الضرورية.

### إنشاء الحزمة الثانية في مساحة العمل (Creating the Second Package in the Workspace)

بعد ذلك، دعونا ننشئ package عضواً آخر في الـ workspace ونسميه `add_one`. قم بتوليد library crate جديد باسم `add_one`:

```console
$ cargo new add_one --lib
     Created library `add_one` package
      Adding `add_one` as member of workspace at `file:///projects/add`
```

سيشمل ملف _Cargo.toml_ في المستوى الأعلى الآن مسار _add_one_ في قائمة `members`:

<span class="filename">اسم الملف: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/Cargo.toml}}
```

يجب أن يحتوي دليل _add_ الخاص بك الآن على هذه الأدلة والملفات:

```text
├── Cargo.lock
├── Cargo.toml
├── add_one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

في ملف _add_one/src/lib.rs_ ، دعونا نضيف function باسم `add_one`:

<span class="filename">اسم الملف: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/add_one/src/lib.rs}}
```

الآن يمكننا جعل package الـ `adder` مع الـ binary الخاص بنا يعتمد على package الـ `add_one` الذي يحتوي على المكتبة الخاصة بنا. أولاً، سنحتاج إلى إضافة اعتماد مسار (path dependency) على `add_one` في ملف _adder/Cargo.toml_.

<span class="filename">اسم الملف: adder/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/adder/Cargo.toml:6:7}}
```

لا يفترض Cargo أن الـ crates في workspace ستعتمد على بعضها البعض، لذا نحتاج إلى أن نكون صريحين بشأن علاقات التبعية (dependency relationships).

بعد ذلك، دعونا نستخدم function الـ `add_one` (من crate الـ `add_one`) في crate الـ `adder`. افتح ملف _adder/src/main.rs_ وغير function الـ `main` لاستدعاء function الـ `add_one` ، كما في القائمة 14-7.

<Listing number="14-7" file-name="adder/src/main.rs" caption="Using the `add_one` library crate from the `adder` crate">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-07/add/adder/src/main.rs}}
```

</Listing>

دعونا نبني الـ workspace عن طريق تشغيل `cargo build` في دليل _add_ في المستوى الأعلى!

```console
$ cargo build
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.22s
```

لتشغيل binary crate من دليل _add_ ، يمكننا تحديد أي package في الـ workspace نريد تشغيله باستخدام وسيط (argument) الـ `-p` واسم الـ package مع `cargo run`:

```console
$ cargo run -p adder
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/adder`
Hello, world! 10 plus one is 11!
```

هذا يشغل الكود في _adder/src/main.rs_ ، والذي يعتمد على crate الـ `add_one`.
```
<a id="depending-on-an-external-package-in-a-workspace"></a>

### الاعتماد على حزمة خارجية (Depending on an External Package)

لاحظ أن الـ workspace يحتوي على ملف _Cargo.lock_ واحد فقط في المستوى الأعلى، بدلاً من وجود _Cargo.lock_ في دليل كل crate. يضمن ذلك أن جميع الـ crates تستخدم نفس الإصدار من جميع التبعيات (dependencies). إذا أضفنا حزمة (package) الـ `rand` إلى ملفات _adder/Cargo.toml_ و _add_one/Cargo.toml_ ، فسيقوم Cargo بحلهما (resolve) إلى إصدار واحد من `rand` وتسجيل ذلك في ملف _Cargo.lock_ الواحد. إن جعل جميع الـ crates في الـ workspace تستخدم نفس الـ dependencies يعني أن الـ crates ستكون دائماً متوافقة مع بعضها البعض. دعونا نضيف crate الـ `rand` إلى قسم `[dependencies]` في ملف _add_one/Cargo.toml_ حتى نتمكن من استخدام crate الـ `rand` في crate الـ `add_one`:

<span class="filename">اسم الملف: add_one/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add/add_one/Cargo.toml:6:7}}
```

يمكننا الآن إضافة `use rand;` إلى ملف _add_one/src/lib.rs_ ، وبناء الـ workspace بالكامل عن طريق تشغيل `cargo build` في دليل _add_ سيجلب ويجمع crate الـ `rand`. سنحصل على تحذير واحد لأننا لا نشير إلى الـ `rand` الذي جلبناه إلى النطاق (scope):

```console
$ cargo build
    Updating crates.io index
  Downloaded rand v0.8.5
   --snip--
   Compiling rand v0.8.5
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
warning: unused import: `rand`
 --> add_one/src/lib.rs:1:5
  |
1 | use rand;
  |     ^^^^
  |
  = note: `#[warn(unused_imports)]` on by default

warning: `add_one` (lib) generated 1 warning (run `cargo fix --lib -p add_one` to apply 1 suggestion)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.95s
```

يحتوي ملف _Cargo.lock_ في المستوى الأعلى الآن على معلومات حول تبعية `add_one` على `rand`. ومع ذلك، على الرغم من استخدام `rand` في مكان ما في الـ workspace، لا يمكننا استخدامه في crates أخرى في الـ workspace ما لم نضف `rand` إلى ملفات _Cargo.toml_ الخاصة بها أيضاً. على سبيل المثال، إذا أضفنا `use rand;` إلى ملف _adder/src/main.rs_ لـ package الـ `adder` ، فسنحصل على خطأ:

```console
$ cargo build
  --snip--
   Compiling adder v0.1.0 (file:///projects/add/adder)
error[E0432]: unresolved import `rand`
 --> adder/src/main.rs:2:5
  |
2 | use rand;
  |     ^^^^ no external crate `rand`
```

لإصلاح ذلك، قم بتحرير ملف _Cargo.toml_ لـ package الـ `adder` وأشر إلى أن `rand` هو تبعية له أيضاً. سيؤدي بناء package الـ `adder` إلى إضافة `rand` إلى قائمة الـ dependencies لـ `adder` في _Cargo.lock_ ، ولكن لن يتم تنزيل نسخ إضافية من `rand`. سيضمن Cargo أن كل crate في كل package في الـ workspace يستخدم package الـ `rand` سيستخدم نفس الإصدار طالما أنها تحدد إصدارات متوافقة من `rand` ، مما يوفر لنا المساحة ويضمن أن الـ crates في الـ workspace ستكون متوافقة مع بعضها البعض.

إذا حددت الـ crates في الـ workspace إصدارات غير متوافقة من نفس التبعية، فسيقوم Cargo بحل كل منها ولكنه سيظل يحاول حل أقل عدد ممكن من الإصدارات.

### إضافة اختبار إلى مساحة عمل (Adding a Test to a Workspace)

لتحسين آخر، دعونا نضيف اختباراً (test) لـ function الـ `add_one::add_one` داخل crate الـ `add_one`:

<span class="filename">اسم الملف: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add/add_one/src/lib.rs}}
```

الآن قم بتشغيل `cargo test` في دليل _add_ في المستوى الأعلى. سيؤدي تشغيل `cargo test` في workspace مهيكل مثل هذا إلى تشغيل الاختبارات لجميع الـ crates في الـ workspace:

```console
$ cargo test
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.20s
     Running unittests src/lib.rs (target/debug/deps/add_one-93c49ee75dc46543)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/adder-3a47283c568d2b6a)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

يوضح القسم الأول من المخرجات أن اختبار `it_works` في crate الـ `add_one` قد نجح. يوضح القسم التالي أنه لم يتم العثور على اختبارات في crate الـ `adder` ، ثم يوضح القسم الأخير أنه لم يتم العثور على اختبارات توثيق (documentation tests) في crate الـ `add_one`.

يمكننا أيضاً تشغيل الاختبارات لـ crate واحد معين في workspace من دليل المستوى الأعلى باستخدام علم (flag) الـ `-p` وتحديد اسم الـ crate الذي نريد اختباره:

```console
$ cargo test -p add_one
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running unittests src/lib.rs (target/debug/deps/add_one-93c49ee75dc46543)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

توضح هذه المخرجات أن `cargo test` قام فقط بتشغيل الاختبارات لـ crate الـ `add_one` ولم يقم بتشغيل اختبارات crate الـ `adder`.

إذا قمت بنشر الـ crates في الـ workspace على [crates.io](https://crates.io/) ، فسيحتاج كل crate في الـ workspace إلى النشر بشكل منفصل. مثل `cargo test` ، يمكننا نشر crate معين في الـ workspace الخاص بنا باستخدام flag الـ `-p` وتحديد اسم الـ crate الذي نريد نشره.

لمزيد من التدريب، أضف crate باسم `add_two` إلى هذا الـ workspace بطريقة مماثلة لـ crate الـ `add_one`!

مع نمو مشروعك، فكر في استخدام workspace: فهو يتيح لك العمل مع مكونات أصغر وأسهل في الفهم من كتلة واحدة كبيرة من الكود. علاوة على ذلك، فإن الاحتفاظ بالـ crates في workspace يمكن أن يجعل التنسيق بين الـ crates أسهل إذا كانت تتغير غالباً في نفس الوقت.
```
