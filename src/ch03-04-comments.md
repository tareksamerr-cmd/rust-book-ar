## التعليقات (Comments)

يسعى جميع المبرمجين لجعل code الخاص بهم سهل الفهم، ولكن في بعض الأحيان يكون هناك ما يبرر وجود شرح إضافي. في هذه الحالات، يترك المبرمجون **تعليقات (comments)** في شفرة المصدر (source code) الخاصة بهم والتي سيتجاهلها المُصرِّف (compiler) ولكن قد يجدها الأشخاص الذين يقرؤون source code مفيدة.

إليك comment بسيط:

```rust
// hello, world
```

في Rust، يبدأ أسلوب comment الاصطلاحي بشرطتين مائلتين، ويستمر comment حتى نهاية السطر. بالنسبة لـ comments التي تمتد إلى ما بعد سطر واحد، ستحتاج إلى تضمين `//` في كل سطر، مثل هذا:

```rust
// So we're doing something complicated here, long enough that we need
// multiple lines of comments to do it! Whew! Hopefully, this comment will
// explain what's going on.
```

يمكن أيضًا وضع comments في نهاية الأسطر التي تحتوي على code:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-24-comments-end-of-line/src/main.rs}}
```

ولكن غالبًا ما ستراها مستخدمة بهذا التنسيق، مع comment على سطر منفصل فوق code الذي تشرحه:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-25-comments-above-line/src/main.rs}}
```

لدى Rust أيضًا نوع آخر من comments، وهو **تعليقات التوثيق (documentation comments)**، والتي سنناقشها في قسم [“نشر حزمة (Crate) إلى Crates.io”][publishing]<!-- ignore --> من الفصل 14.

[publishing]: ch14-02-publishing-to-crates-io.html
