# ابدأ من هنا

## 1) جرّب بلا روم وبلا هاتف — دقيقة واحدة

```bash
chmod +x colorport
./colorport demo
```

يبني شجرتين صغيرتين (جهاز وهمي + ColorOS وهمي) ويمرّ بك على الدورة كاملة:
فحص → خطّة → تجربة → تنفيذ → تحقق → قواعد SELinux → تراجع، ثم يقول هل عادت الشجرة
مطابقة للأصل بالبايت. هذه ليست محاكاة: هي نفس الدوال التي تعمل على رومك.

ثم الواجهة:

```bash
./colorport ui        # http://127.0.0.1:8765
```

## 2) حضّر شجرة رومك

**من الهاتف (بلا حاسوب):**
1. نزّل روم جهازك (نفس إصدار نظامك الحالي).
2. افتحه بـ**MT Manager** → فكّ `payload.bin` → استخرج `system.img` و`vendor.img` →
   استخرج مجلداتهما.
3. رتّبها هكذا: `.../root/system` و`.../root/vendor` (وهكذا لبقية الأقسام).

**على Linux (أو داخل Codespace):** تحتاج أدوات الفك (`payload-dumper-go`، `erofs-utils`،
`simg2img`…). الأداة لا تفكّ الأرشيف بنفسها؛ جهّز الشجرة المفكوكة ثم أعطها المسار.

## 3) افحص أولاً — لا تعدّل شيئاً

```bash
./colorport scan /مسار/root
```

اقرأ `out/scan.md`: أي HAL معلن، أي بلوبات موجودة، وأي مجال `found` أو `missing` أو
`partial`. لا شيء تغيّر بعد.

## 4) ابنِ الخطّة

```bash
./colorport plan --target /مسار/root --source /مسار/coloros/root
```

اقرأ `out/plan.md` وابحث عن:

| ما تراه | معناه | ما تفعله |
| --- | --- | --- |
| `تلقائي (نسخ ملفات)` | ملفات تُنسخ وحدها | راجع ما سيُنقل |
| `بمساعدة (قطعة من جهازك)` | يحتاج بلوباً/معايرة من جهازك | اقرأ السطر: يسمّي القطعة |
| `يدوي (تعديل إطار/تُختبر بالاستعمال)` | لا ملف يحكم فيه | اختبره على الجهاز |
| `HAL يُدمج في manifest جهازك` | إعلان HAL ناقص | يُضاف داخل manifest جهازك |
| `جاهز (كل قطعه موجودة)` | القطع موجودة، لا أن الميزة تعمل | افتح الكاميرا/البصمة واحكم |

## 5) نفّذ — تجربة أولاً دائماً

```bash
./colorport apply --target /مسار/root --source /مسار/coloros/root            # قراءة فقط
./colorport apply --target /مسار/root --source /مسار/coloros/root --write --props
./colorport verify /مسار/root
./colorport coverage
```

- كل تعديل مسجَّل في `.colorport/manifest.json` ومعه نسخة احتياطية.
- **قبل أول تنفيذ خذ نسخة**: `cp -a root root.backup`.
- أخطأت؟ `./colorport uninstall /مسار/root` يعيد الشجرة.

## 6) SELinux بعد أول إقلاع فقط

```bash
adb shell 'dmesg | grep avc' > denials.txt
adb logcat -d | grep avc >> denials.txt
./colorport sepolicy --denials denials.txt --rom /مسار/root
./colorport sepolicy --denials denials.txt --rom /مسار/root --install
```

أعد التفليش وأعد الخطوة حتى لا يبقى `avc: denied` جديد. لا تستخدم permissive إلا
لإقلاع واحد لالتقاط السجلات.

## 7) ما لا تفعله

- لا تنسخ `manifest.xml` من روم آخر بيدك: الأداة تدمجه وتحفظ HALs جهازك.
- لا تعتمد على props جهاز آخر: القيم تُقرأ من جهازك.
- لا تفلش بلا نسخة كاملة قابلة للرجوع.
- لا تسأل «لماذا لا تعمل؟» قبل قراءة [الحدود](LIMITS.ar.md).
