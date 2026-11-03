# MUPS Survey System — Supabase + Vercel

Энэ багц нь MUPS Survey System-ийн браузер доторх localStorage демо бүртгэлийг Supabase Authentication болон PostgreSQL өгөгдлийн сантай холбох үйлдвэрлэлийн суурь хувилбар юм.

## Багцын файлууд

- `index.html` — веб аппликейшн
- `config.js` — Supabase төслийн URL болон browser-д ашиглах Publishable key
- `config.example.js` — тохиргооны жишээ
- `supabase_schema.sql` — хүснэгт, trigger, индекс, Row Level Security (RLS) бодлого
- `vercel.json` — Vercel-ийн үндсэн тохиргоо

## 1. Supabase төслөө үүсгэх

1. https://supabase.com руу орж шинэ project үүсгэнэ.
2. Project-ийн **SQL Editor** хэсгийг нээнэ.
3. `supabase_schema.sql` файлын бүх агуулгыг хуулж SQL Editor-д ажиллуулна.
4. **Project Settings → API** хэсгээс Project URL болон Publishable key-г авна.
5. `config.js` файлын `supabaseUrl`, `supabasePublishableKey` утгуудыг солино.

> Publishable key нь browser-д ашиглах зориулалттай. `service_role` эсвэл secret key-г `config.js`, HTML, GitHub, чатад хэзээ ч хийж болохгүй.

## 2. Authentication тохиргоо

Supabase → **Authentication → URL Configuration** хэсэгт:

- Site URL: Vercel-ээс олгосон үндсэн домэйн
- Redirect URLs: үндсэн домэйн болон шаардлагатай preview домэйнууд

Эхний туршилтад **Authentication → Providers → Email** идэвхтэй байхад хангалттай. И-мэйл баталгаажуулалт идэвхтэй бол бүртгүүлсний дараа хэрэглэгч и-мэйлээ баталгаажуулна.

## 3. Судлаачийн эрх олгох

Аюулгүй байдлын үүднээс бүртгүүлсэн хүн өөрөө `researcher` эрх сонгож авах боломжгүй. Шинэ бүртгэл `participant` эрхтэй үүснэ.

Хэрэглэгч системд бүртгүүлсний дараа Supabase → SQL Editor дээр дараах командыг тухайн хүний бодит и-мэйлээр ажиллуулж судлаач болгоно:

```sql
update public.profiles
set role = 'researcher', updated_at = now()
where id = (
  select id from auth.users where email = 'researcher@example.edu'
);
```

`researcher@example.edu`-г бодит хэрэглэгчийн и-мэйлээр солино. Энэ командыг зөвхөн төслийн эрх бүхий администратор ажиллуулна.

## 4. Vercel-д байрлуулах

1. `index.html`, `config.js`, `vercel.json` файлуудыг GitHub repository-д байрлуулна.
2. https://vercel.com дээр **Add New → Project** сонгож repository-гоо холбоно.
3. Framework Preset: **Other**; Build Command болон Output Directory-г хоосон/анхдагч хэвээр үлдээнэ.
4. Deploy хийнэ.
5. Vercel домэйныг Supabase Authentication URL Configuration-д Site URL болон Redirect URL болгон нэмнэ.

`config.js` нь publishable key агуулдаг тул GitHub-д байрлуулах нь Supabase-ийн зөв client-side хэрэглээ юм. Харин database-ийн хамгаалалт RLS бодлогоор хийгдэнэ.

## 5. Одоогийн ажиллагаа

- Supabase Auth ашиглан бүртгүүлэх, нэвтрэх, гарах
- Шинэ хэрэглэгчийг автоматаар `participant` профайлтай үүсгэх
- Администраторын гараар судлаачийн эрх олгох
- Судлаачийн өөрийн асуумжийг серверт хадгалах, нээх/хаах
- Нээлттэй асуумжийг нэвтэрсэн оролцогчид харуулах
- Хариултыг PostgreSQL-д хадгалах; давхар бөглөлтийг нэг хэрэглэгч/нэг асуумжийн хослолоор хориглох
- Судалгааны эзэмшигч зөвхөн өөрийн судалгааны хариултыг харах
- Олон төхөөрөмжөөс ижил холбоосоор нээлттэй асуумж руу орох
- Одоо байгаа давтамж, тодорхойлох статистик, Кронбахын альфа, Пирсоны корреляци, XLSX/Word/PDF тайлангийн функцуудыг хадгалах

## Анхаарах зүйлс

- Одоо бүртгэгдсэн localStorage демо хэрэглэгч, асуумж, хариултыг сервер рүү автоматаар шилжүүлэхгүй. Хуучин өгөгдлийг бодит хэрэглээнд оруулах бол эзэмшил, зөвшөөрлийг шалгасан тусгай импорт шаардлагатай.
- Оролцогч заавал бүртгүүлж, нэвтэрсний дараа асуумж бөглөнө. Нэргүй/нийтийн хариултын горим энэ багцад ороогүй.
- Судалгааны өгөгдөлд эмзэг эсвэл таних боломжтой мэдээлэл цуглуулах бол байгууллагын ёс зүй, зөвшөөрөл, хадгалалтын хугацаа, нууцлалын шаардлагыг тусад нь баталгаажуулна.
- Энд бэлтгэсэн багц нь төслийн Supabase/Vercel дансанд өөрөө нэвтэрч байршуулсан гэсэн үг биш. Бодит URL болон project тохиргоог эзэмшигч тал хийх шаардлагатай.
