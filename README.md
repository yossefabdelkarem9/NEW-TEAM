# NEW TEAM Store

متجر إلكتروني عربي متجاوب لعرض وبيع الكوتشيات والجزم والسليبرز، مع سلة مشتريات، تنفيذ طلبات، إدارة مخزون، ولوحة تحكم للأدمن.

> **الحالة:** مشروع Frontend يعمل كصفحات HTML ثابتة، مع Firebase كخدمة Backend بدون خادم مخصص.

## نظرة سريعة

المشروع يتكون من واجهتين:

- **واجهة المتجر** في [`index.html`](./index.html)
  - عرض المنتجات من Firestore.
  - البحث والتصفية حسب القسم والمقاس.
  - معاينة الألوان والصور والمقاسات.
  - سلة مشتريات محفوظة في `localStorage`.
  - تنفيذ طلب توصيل أو استلام من الفرع.
  - وضع فاتح وداكن محفوظ للمستخدم.
  - قسم صور Editorial للهوية البصرية.
  - رابط واتساب ثابت في Footer الصفحة.
- **لوحة الإدارة** في [`admin.html`](./admin.html)
  - تسجيل دخول Firebase Authentication.
  - إضافة وتعديل وحذف المنتجات.
  - رفع صور الألوان إلى Cloudinary.
  - إدارة المقاسات والكميات لكل لون.
  - متابعة الطلبات وتحديث حالتها.
  - مؤشرات المنتجات والطلبات والمبيعات.

## المعمارية

```text
Browser
├── index.html
│   ├── Tailwind CDN + Custom CSS
│   ├── Firebase Authentication (Anonymous customer session)
│   ├── Firestore: products / orders
│   └── localStorage: shopping cart + theme preference
│
├── admin.html
│   ├── Firebase Authentication (admin email/password)
│   ├── Firestore realtime listeners
│   └── Cloudinary Upload Widget
│
├── firebase-config.js
│   ├── Firebase project configuration
│   ├── Cloudinary configuration
│   └── Store contact and branch information
│
├── firestore.rules
└── assets/ + images/
    ├── SVG brand logos
    └── Editorial brand photography
```

لا يوجد build system أو framework إجباري حاليًا؛ الصفحات تعمل مباشرة عبر Static Hosting، وتستخدم ES Modules من Firebase CDN.

## التقنيات

| المجال | التقنية |
|---|---|
| UI | HTML5, CSS3, Tailwind CSS CDN |
| اللغة والاتجاه | Arabic / RTL |
| الخط | Cairo من Google Fonts |
| البيانات | Cloud Firestore |
| المصادقة | Firebase Authentication |
| رفع الصور | Cloudinary Upload Widget |
| تخزين السلة | Browser localStorage |
| التوافق | Responsive desktop/tablet/mobile |
| الهوية | SVG logo + lime/black brand palette |

## ملفات المشروع

| الملف | المسؤولية |
|---|---|
| [`index.html`](./index.html) | واجهة المتجر، الفلاتر، السلة، Checkout |
| [`admin.html`](./admin.html) | تسجيل الدخول ولوحة المنتجات والطلبات |
| [`firebase-config.js`](./firebase-config.js) | إعداد Firebase وCloudinary وبيانات الفرع |
| [`firestore.rules`](./firestore.rules) | قواعد القراءة والكتابة في Firestore |
| [`assets/new-team-logo.svg`](./assets/new-team-logo.svg) | شعار الوضع الفاتح |
| [`assets/new-team-logo-light.svg`](./assets/new-team-logo-light.svg) | شعار الخلفيات الداكنة |
| [`images/`](./images/) | صور الهوية البصرية وEditorial Showcase |

## نموذج البيانات

### `products`

```js
{
  name: "Air Force",
  category: "كوتشيات",
  price: 500,
  discountPrice: 399,
  description: "وصف المنتج",
  variants: [
    {
      colorName: "ابيض",
      colorHex: "#ffffff",
      images: ["https://..."],
      sizes: {
        "38": 2,
        "39": 4,
        "40": 0,
        "41": 3
      }
    }
  ],
  createdAt: Timestamp
}
```

### `orders`

```js
{
  customerName: "اسم العميل",
  phone: "رقم العميل",
  deliveryMethod: "delivery", // delivery | pickup
  address: "العنوان أو عنوان الفرع",
  items: [
    {
      productId: "...",
      name: "Air Force",
      color: "ابيض",
      size: "41",
      qty: 1,
      price: 399
    }
  ],
  totalAmount: 399,
  status: "pending",
  createdAt: Timestamp
}
```

حالات الطلب المستخدمة في لوحة الإدارة:

- `pending`
- `confirmed`
- `shipped`
- `completed`
- `cancelled`

## تدفق تنفيذ الطلب

1. العميل يختار المنتج واللون والمقاس.
2. المنتج يضاف إلى السلة مع `productId` وكمية المخزون المتاحة.
3. السلة تحفظ محليًا في `localStorage` تحت المفتاح `stepx_cart`.
4. عند التأكيد يتم تشغيل Firebase Anonymous Authentication.
5. يتم تشغيل Firestore Transaction:
   - قراءة المنتجات الحالية.
   - التحقق من توفر المخزون.
   - خصم الكمية ذريًا.
   - إنشاء مستند جديد داخل `orders`.
6. بعد النجاح يتم تفريغ السلة وإعادة تحميل المنتجات.

## الصلاحيات وقواعد Firestore

القواعد موجودة في [`firestore.rules`](./firestore.rules).

- قراءة المنتجات متاحة للزوار.
- كتابة المنتجات متاحة للمستخدم المصادق عليه فقط.
- إنشاء الطلب يتطلب جلسة Firebase Authentication.
- إنشاء الطلب يقبل الحقول الأساسية فقط بشروط نوعية أولية.
- قراءة وتعديل الطلبات مخصصة للمستخدمين المصادق عليهم.

### إعداد مطلوب قبل تشغيل الطلبات

من Firebase Console:

1. افتح **Authentication → Sign-in method**.
2. فعّل **Anonymous**.
3. افتح **Firestore Database → Rules**.
4. انشر محتوى [`firestore.rules`](./firestore.rules).

> في بيئة إنتاج حقيقية، يفضل نقل خصم المخزون وإنشاء الطلب إلى Cloud Function أو Backend موثوق، مع قواعد أكثر تقييدًا للأدمن وتحقق أعمق من الأسعار والمنتجات.

## إعدادات المتجر

يتم تعديل العنوان ورقم التواصل وساعات العمل من [`firebase-config.js`](./firebase-config.js):

```js
export const storeInfo = {
  brandName: "NEW TEAM STORE",
  branchAddress: "سوهاج - طما - شارع الجمهورية - امام كشري السلطان",
  workingHours: "يومياً من 12 ظهراً حتى 11 مساءً",
  phoneNumber: "01028060055",
  whatsappNumber: "201028060055"
};
```

يتم استخدام البيانات تلقائيًا في:

- بيانات الاستلام من الفرع.
- رابط الاتصال.
- رابط واتساب في أسفل المتجر.

## الهوية والتصميم

تم بناء التصميم حول هوية NEW TEAM:

- Lime accent: `#C8FF35`
- Dark background: `#0B0D10`
- Dark surface: `#15181D`
- Secondary dark surface: `#20242B`
- Main light text: `#F5F7FA`
- Muted text: `#A7AFBC`

### Dark Mode

- الاختيار يحفظ في `localStorage` تحت المفتاح `new-team-theme`.
- الشعار يتبدل بين نسخة الخلفية الفاتحة ونسخة الخلفية الداكنة.
- يتم ضبط ألوان النصوص والكروت والحدود والـ badges والحقول بشكل مستقل.
- الهيدر والسلة والنوافذ المنبثقة لها ألوان متوافقة مع الثيم.

## الصور ورفع المنتجات

صور المنتجات الجديدة ترفع من لوحة الإدارة عن طريق Cloudinary Upload Widget.

إعداد Cloudinary موجود في [`firebase-config.js`](./firebase-config.js):

```js
export const cloudinaryConfig = {
  cloudName: "pazsav4q",
  uploadPreset: "NEW TEAM"
};
```

أما صور الهوية الثابتة فتوجد داخل [`images/`](./images/) ويتم عرضها في قسم Editorial مستقل، ولا تختلط مع صور المنتجات القادمة من Firestore.

## التشغيل المحلي

لأن المشروع يستخدم ES Modules وFirebase، يفضل تشغيله عبر HTTP server وليس `file://`.

### Python

```bash
python3 -m http.server 5500
```

ثم افتح:

```text
http://localhost:5500/index.html
http://localhost:5500/admin.html
```

### VS Code Live Server

يمكن تشغيل `index.html` أو `admin.html` باستخدام إضافة Live Server.

## النشر

المشروع مناسب للنشر على:

- GitHub Pages
- Firebase Hosting
- Netlify
- Vercel Static Hosting

عند النشر تأكد من:

1. إضافة الدومين إلى Firebase Authentication authorized domains.
2. نشر `firestore.rules`.
3. تفعيل Anonymous Authentication.
4. ضبط صلاحيات Cloudinary Upload Preset.
5. عدم وضع أسرار إدارية أو مفاتيح Server SDK داخل ملفات الواجهة.

## الاختبار اليدوي

### المتجر

- فتح الصفحة على Desktop وMobile.
- تبديل الوضع الفاتح والداكن.
- البحث باسم منتج.
- فلترة القسم.
- فلترة المقاس.
- فتح تفاصيل المنتج.
- تبديل اللون والصورة.
- اختيار مقاس متوفر.
- إضافة المنتج للسلة.
- تعديل الكمية وحذف المنتج.
- تجربة التوصيل والاستلام من الفرع.
- التأكد من رابط واتساب في أسفل الصفحة.

### لوحة الإدارة

- فتح الصفحة على شاشة بعرض هاتف.
- التأكد من عدم وجود horizontal overflow.
- تسجيل الدخول.
- إضافة منتج.
- رفع صور لون.
- تحديد المقاسات والمخزون.
- تعديل وحذف منتج.
- فتح الطلبات وتغيير الحالة.
- تبديل الوضع الليلي.

## ملاحظات أمنية وتشغيلية

- `firebase-config.js` يحتوي إعدادات العميل العامة، وليس بديلًا عن Firestore Security Rules.
- لا تضع Firebase Admin SDK أو Cloudinary API Secret في الواجهة.
- يفضل إضافة تحقق Server-side من الأسعار والمخزون قبل اعتماد الطلب النهائي.
- يفضل تقييد `products` write إلى حسابات الأدمن فقط بدل أي مستخدم مصادق عليه.
- يفضل إضافة rate limiting أو App Check لتقليل الطلبات الآلية.

## تحسينات مستقبلية مقترحة

1. Cloud Functions لمعالجة الطلبات وخصم المخزون بشكل آمن.
2. App Check لحماية Firebase من الاستخدام غير المصرح.
3. Wishlist وحفظ المنتجات المفضلة.
4. كوبونات وعروض موسمية.
5. تقييمات العملاء.
6. تقارير مبيعات متقدمة.
7. إشعارات WhatsApp أو البريد عند إنشاء الطلب.
8. صفحة مستقلة لكل منتج وتحسين SEO.
9. تحويل Tailwind CDN إلى build pipeline للإنتاج.
10. إضافة اختبارات آلية لتدفق السلة والـ checkout.
