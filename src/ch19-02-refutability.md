## قابلية النقض: عندما قد يفشل النمط في المطابقة (Refutability: Whether a Pattern Might Fail to Match)

تأتي الأنماط (Patterns) في شكلين: قابلة للنقض (Refutable) وغير قابلة للنقض (Irrefutable). الأنماط التي ستطابق أي قيمة محتملة يتم تمريرها هي أنماط _Irrefutable_. مثال على ذلك سيكون `x` في العبارة `let x = 5;` لأن `x` يطابق أي شيء وبالتالي لا يمكن أن يفشل في المطابقة. أما الأنماط التي يمكن أن تفشل في المطابقة لبعض القيم المحتملة فهي أنماط _Refutable_. مثال على ذلك سيكون `Some(x)` في التعبير `if let Some(x) = a_value` لأنه إذا كانت القيمة في المتغير (Variable) `a_value` هي `None` بدلاً من `Some` ، فإن نمط `Some(x)` لن يطابق.

يمكن لوسائط الدوال (Function Parameters)، وعبارات `let` ، وحلقات (Loops) الـ `for` أن تقبل فقط Patterns من نوع Irrefutable لأن البرنامج لا يمكنه فعل أي شيء ذي معنى عندما لا تتطابق القيم. تقبل تعبيرات `if let` و `while let` وعبارة `let...else` كلاً من Patterns الـ Refutable والـ Irrefutable، لكن المترجم (Compiler) يحذر من استخدام Irrefutable Patterns لأنها، بحكم تعريفها، مخصصة للتعامل مع الفشل المحتمل: تكمن وظيفة الشرط في قدرته على الأداء بشكل مختلف اعتماداً على النجاح أو الفشل.

بشكل عام، لا ينبغي أن تقلق بشأن التمييز بين Refutable Patterns و Irrefutable Patterns؛ ومع ذلك، فأنت بحاجة إلى التعرف على مفهوم قابلية النقض (Refutability) حتى تتمكن من الاستجابة عندما تراها في رسالة خطأ. في تلك الحالات، ستحتاج إلى تغيير إما Pattern أو البنية التي تستخدم Pattern معها، اعتماداً على السلوك المقصود من الكود (Code).

دعونا نلقي نظرة على مثال لما يحدث عندما نحاول استخدام Refutable Pattern حيث تتطلب لغة Rust استخدام Irrefutable Pattern والعكس صحيح. توضح القائمة (Listing) 19-8 عبارة `let` ، ولكن بالنسبة لـ Pattern ، فقد حددنا `Some(x)` ، وهو Refutable Pattern. وكما قد تتوقع، لن يتم تحويل هذا الكود برمجياً (Compile).

<Listing number="19-8" caption="محاولة استخدام نمط قابل للنقض مع `let` ">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-08/src/main.rs:here}}
```

</Listing>

إذا كانت `some_option_value` قيمة `None` ، فستفشل في مطابقة Pattern `Some(x)` ، مما يعني أن Pattern هو Refutable. ومع ذلك، يمكن لعبارة `let` أن تقبل فقط Irrefutable Pattern لأنه لا يوجد شيء صالح يمكن لـ Code القيام به مع قيمة `None`. في وقت التحويل البرمجي (Compile Time)، ستشتكي Rust من أننا حاولنا استخدام Refutable Pattern حيث يلزم استخدام Irrefutable Pattern:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-08/output.txt}}
```

لأننا لم نغطِ (ولم نتمكن من تغطية!) كل قيمة صالحة باستخدام Pattern `Some(x)` ، فإن Rust تنتج بشكل محق خطأ في Compiler.

إذا كان لدينا Refutable Pattern حيث يلزم وجود Irrefutable Pattern ، فيمكننا إصلاح ذلك عن طريق تغيير Code الذي يستخدم Pattern: بدلاً من استخدام `let` ، يمكننا استخدام `let...else`. بعد ذلك، إذا لم يتطابق Pattern ، فسيقوم Code الموجود بين الأقواس المتعرجة بالتعامل مع القيمة. توضح Listing 19-9 كيفية إصلاح Code في Listing 19-8.

<Listing number="19-9" caption="استخدام `let...else` وكتلة برمجية مع أنماط قابلة للنقض بدلاً من `let` ">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-09/src/main.rs:here}}
```

</Listing>

لقد أعطينا لـ Code مخرجاً! هذا الكود صالح تماماً، على الرغم من أنه يعني أننا لا نستطيع استخدام Irrefutable Pattern دون تلقي تحذير. إذا أعطينا `let...else` نمطاً سيطابق دائماً، مثل `x` ، كما هو موضح في Listing 19-10، فسيقوم Compiler بإعطاء تحذير.

<Listing number="19-10" caption="محاولة استخدام نمط غير قابل للنقض مع `let...else` ">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-10/src/main.rs:here}}
```

</Listing>

تشتكي Rust من أنه ليس من المنطقي استخدام `let...else` مع Irrefutable Pattern:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-10/output.txt}}
```

لهذا السبب، يجب أن تستخدم أذرع المطابقة (Match Arms) أنماطاً من نوع Refutable ، باستثناء الذراع الأخير، الذي يجب أن يطابق أي قيم متبقية باستخدام Irrefutable Pattern. تسمح لنا Rust باستخدام Irrefutable Pattern في `match` بذراع واحد فقط، لكن هذه الصيغة ليست مفيدة بشكل خاص ويمكن استبدالها بعبارة `let` أبسط.

الآن بعد أن عرفت أين تستخدم Patterns والفرق بين Refutable Patterns و Irrefutable Patterns ، فلنغطِ كل الصيغ التي يمكننا استخدامها لإنشاء Patterns.
