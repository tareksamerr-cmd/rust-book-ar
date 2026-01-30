## الملحق أ: الكلمات المفتاحية (Keywords)

تحتوي القوائم التالية على "الكلمات المفتاحية" (Keywords) المحجوزة للاستخدام الحالي أو المستقبلي بواسطة لغة Rust. وبناءً على ذلك، لا يمكن استخدامها كـ "معرفات" (Identifiers) (باستثناء استخدامها كـ "معرفات خام" - Raw Identifiers، كما سنناقش في قسم [المعرفات الخام][raw-identifiers]). الـ Identifiers هي أسماء الدوال، والمتغيرات، والمعاملات، وحقول الهياكل (Struct Fields)، والوحدات (Modules)، والحزم (Crates)، والثوابت، والماكرو (Macros)، والقيم الساكنة، والسمات (Attributes)، والأنواع، والسمات (Traits)، أو فترات الحياة (Lifetimes).

[raw-identifiers]: #raw-identifiers

### الكلمات المفتاحية المستخدمة حالياً

فيما يلي قائمة بالـ Keywords المستخدمة حالياً، مع وصف لوظائفها.

- **`as`**: إجراء "تحويل أولي" (Primitive Casting)، أو إزالة الغموض عن Trait معين يحتوي على عنصر، أو إعادة تسمية العناصر في عبارات `use`.
- **`async`**: إرجاع `Future` بدلاً من حظر المسار (Thread) الحالي.
- **`await`**: تعليق التنفيذ حتى تصبح نتيجة الـ `Future` جاهزة.
- **`break`**: الخروج من حلقة تكرار (Loop) فوراً.
- **`const`**: تعريف عناصر ثابتة أو مؤشرات خام (Raw Pointers) ثابتة.
- **`continue`**: الاستمرار إلى التكرار التالي للحلقة.
- **`crate`**: في مسار الوحدة، يشير إلى جذر الـ Crate.
- **`dyn`**: "إرسال ديناميكي" (Dynamic Dispatch) إلى كائن سمة (Trait Object).
- **`else`**: خيار احتياطي لهياكل تدفق التحكم `if` و `if let`.
- **`enum`**: تعريف "تعداد" (Enumeration).
- **`extern`**: ربط دالة أو متغير خارجي.
- **`false`**: القيمة المنطقية "خطأ".
- **`fn`**: تعريف دالة أو نوع مؤشر دالة.
- **`for`**: التكرار عبر عناصر من مكرر (Iterator)، أو تنفيذ Trait، أو تحديد Lifetime ذات رتبة أعلى.
- **`if`**: التفرع بناءً على نتيجة تعبير شرطي.
- **`impl`**: تنفيذ وظائف ذاتية أو وظائف Trait.
- **`in`**: جزء من صيغة حلقة `for`.
- **`let`**: ربط متغير.
- **`loop`**: التكرار دون قيد أو شرط.
- **`match`**: مطابقة قيمة مع أنماط (Patterns).
- **`mod`**: تعريف Module.
- **`move`**: جعل "الإغلاق" (Closure) يأخذ ملكية جميع ما يلتقطه.
- **`mut`**: الإشارة إلى "القابلية للتغيير" (Mutability) في المراجع، أو الـ Raw Pointers، أو روابط الأنماط.
- **`pub`**: الإشارة إلى الرؤية العامة في Struct Fields، أو كتل `impl` أو الـ Modules.
- **`ref`**: الربط عن طريق المرجع.
- **`return`**: الإرجاع من الدالة.
- **`Self`**: اسم مستعار للنوع الذي نقوم بتعريفه أو تنفيذه.
- **`self`**: موضوع الدالة المرتبطة (Method) أو الـ Module الحالي.
- **`static`**: متغير عام أو Lifetime تستمر طوال فترة تنفيذ البرنامج.
- **`struct`**: تعريف هيكل (Structure).
- **`super`**: الـ Module الأب للـ Module الحالي.
- **`trait`**: تعريف Trait.
- **`true`**: القيمة المنطقية "صواب".
- **`type`**: تعريف اسم مستعار للنوع (Type Alias) أو نوع مرتبط (Associated Type).
- **`union`**: تعريف [اتحاد][union]؛ وتعتبر Keyword فقط عند استخدامها في إعلان الاتحاد.
- **`unsafe`**: الإشارة إلى كود، أو دوال، أو Traits، أو عمليات تنفيذ غير آمنة.
- **`use`**: جلب الرموز إلى النطاق (Scope).
- **`where`**: الإشارة إلى البنود التي تقيد النوع.
- **`while`**: التكرار المشروط بناءً على نتيجة تعبير.

[union]: ../reference/items/unions.html

### الكلمات المفتاحية المحجوزة للاستخدام المستقبلي

الـ Keywords التالية لا تمتلك أي وظيفة بعد ولكنها محجوزة بواسطة Rust للاستخدام المستقبلي المحتمل:

- `abstract`
- `become`
- `box`
- `do`
- `final`
- `gen`
- `macro`
- `override`
- `priv`
- `try`
- `typeof`
- `unsized`
- `virtual`
- `yield`

### المعرفات الخام (Raw Identifiers)

"المعرفات الخام" (Raw Identifiers) هي الصيغة التي تسمح لك باستخدام الـ Keywords في الأماكن التي لا يُسمح فيها عادةً باستخدامها. تستخدم الـ Raw Identifier عن طريق إضافة البادئة `r#` قبل الـ Keyword.

على سبيل المثال، `match` هي Keyword. إذا حاولت تجميع الدالة التالية التي تستخدم `match` كاسم لها:

<span class="filename">اسم الملف: src/main.rs</span>

```rust,ignore,does_not_compile
fn match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}
```

ستحصل على هذا الخطأ:

```text
error: expected identifier, found keyword `match`
 --> src/main.rs:4:4
  |
4 | fn match(needle: &str, haystack: &str) -> bool {
  |    ^^^^^ expected identifier, found keyword
```

يوضح الخطأ أنه لا يمكنك استخدام الـ Keyword `match` كـ Identifier للدالة. لاستخدام `match` كاسم دالة، تحتاج إلى استخدام صيغة الـ Raw Identifier، هكذا:

<span class="filename">اسم الملف: src/main.rs</span>

```rust
fn r#match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}

fn main() {
    assert!(r#match("foo", "foobar"));
}
```

سيتم تجميع هذا الكود دون أي أخطاء. لاحظ البادئة `r#` في اسم الدالة في تعريفها وكذلك في مكان استدعاء الدالة في `main`.

تسمح لك الـ Raw Identifiers باستخدام أي كلمة تختارها كـ Identifier، حتى لو كانت تلك الكلمة Keyword محجوزة. يمنحنا هذا مزيداً من الحرية في اختيار أسماء الـ Identifiers، كما يسمح لنا بالتكامل مع البرامج المكتوبة بلغة لا تعتبر هذه الكلمات Keywords فيها. بالإضافة إلى ذلك، تسمح لك الـ Raw Identifiers باستخدام مكتبات مكتوبة بإصدار (Edition) مختلف من Rust عن الذي تستخدمه الـ Crate الخاصة بك. على سبيل المثال، `try` ليست Keyword في إصدار 2015 ولكنها كذلك في إصدارات 2018 و2021 و2024. إذا كنت تعتمد على مكتبة مكتوبة باستخدام إصدار 2015 وبها دالة `try` فستحتاج إلى استخدام صيغة الـ Raw Identifier، وهي `r#try` في هذه الحالة، لاستدعاء تلك الدالة من الكود الخاص بك في الإصدارات اللاحقة. راجع [الملحق هـ][appendix-e] لمزيد من المعلومات حول الإصدارات.

[appendix-e]: appendix-05-editions.html
