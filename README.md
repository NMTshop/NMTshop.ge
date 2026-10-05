# NMT Shop — გამართვის ინსტრუქცია

ეს საიტი შედგება სამი ფაილისგან:

- `index.html` — საჯარო გვერდი, სადაც მომხმარებლები ხედავენ პროდუქციას
- `admin.html` — პაროლით დაცული გვერდი, სადაც თქვენ მართავთ პროდუქციას
- `config.js` — კავშირი თქვენს Firebase მონაცემთა ბაზასთან
- `style.css` — დიზაინი

პროდუქციის მონაცემები ინახება **Firebase**-ში (Google-ის უფასო სერვისი), ასე რომ როცა ადმინ-პანელში რამეს დაამატებთ ან შეცვლით, ეს მყისიერად აისახება საჯარო გვერდზეც — ყველასთვის.

---

## ნაბიჯი 1 — Firebase პროექტის შექმნა (უფასო)

1. გახსენით [console.firebase.google.com](https://console.firebase.google.com) და შედით თქვენი Google ანგარიშით.
2. დააჭირეთ **„Add project"** და დაარქვით, მაგ. `nmtshop`.
3. Google Analytics შეგიძლიათ გამორთოთ (არ გჭირდებათ) — დააჭირეთ **„Create project"**.

### ჩართეთ სამი სერვისი:

**Firestore Database** (სადაც ინახება პროდუქტები):
- მარცხენა მენიუში: Build → Firestore Database → **Create database**
- აირჩიეთ **„Start in production mode"** → Next → აირჩიეთ ყველაზე ახლოს მდებარე რეგიონი → Enable

**Authentication** (ადმინის შესასვლელი):
- Build → Authentication → **Get started**
- Sign-in method ჩანართში ჩართეთ **Email/Password**
- Users ჩანართში დააჭირეთ **Add user** და შექმენით საკუთარი ადმინის მეილი + პაროლი — ეს არის ის მონაცემები, რითაც `admin.html`-ზე შეხვალთ.

**Storage** (სადაც ინახება სურათები):
- Build → Storage → **Get started** → Next → აირჩიეთ იგივე რეგიონი → Done

### აიღეთ კონფიგურაცია:
- დააჭირეთ ⚙️ (Project settings) → ჩამოსქროლეთ **„Your apps"**-მდე → აირჩიეთ `</>` (Web)
- დაარქვით აპს სახელი (მაგ. `nmtshop-web`) → Register app
- დაკოპირეთ `firebaseConfig` ობიექტის მნიშვნელობები (`apiKey`, `authDomain` და ა.შ.) და ჩასვით `config.js` ფაილში, placeholder-ების ნაცვლად.

---

## ნაბიჯი 2 — უსაფრთხოების წესები (Security Rules)

ნაგულისხმევად Production mode-ში ყველაფერი დაბლოკილია. საჭიროა წესების დაყენება: ყველას შეუძლია **წაკითხვა**, მხოლოდ შესულ ადმინს — **წერა**.

**Firestore Rules** (Firestore Database → Rules):
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /products/{productId} {
      allow read: if true;
      allow write: if request.auth != null;
    }
  }
}
```

**Storage Rules** (Storage → Rules):
```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /products/{allPaths=**} {
      allow read: if true;
      allow write: if request.auth != null;
    }
  }
}
```

ორივეგან დააჭირეთ **Publish**.

---

## ნაბიჯი 3 — ადგილობრივად შემოწმება

გახსენით `index.html` ბრაუზერში (ორჯერ დააწკაპუნეთ ფაილზე) — უნდა ჩანდეს „ჯერ არცერთი პროდუქტი არ დამატებულა". შემდეგ გახსენით `admin.html`, შედით თქვენი მეილით და პაროლით, დაამატეთ ტესტ-პროდუქტი — `index.html` განახლებით უნდა გამოჩნდეს.

> თუ კონსოლში (F12) ხედავთ `auth/unauthorized-domain` შეცდომას ადგილობრივად გახსნისას — ეს ნორმალურია, ჰოსტინგზე ატვირთვის შემდეგ გაქრება (იხ. ნაბიჯი 5).

---

## ნაბიჯი 4 — ჰოსტინგის არჩევა და ატვირთვა

სამივე ქვემოთჩამოთვლილი უფასოა და მხარდაჭერს თქვენს საკუთარ დომენს:

| ვარიანტი | სირთულე | რეკომენდაცია |
|---|---|---|
| **Firebase Hosting** | მარტივი, იგივე ეკოსისტემა რასაც უკვე იყენებთ | ✅ ყველაზე მოსახერხებელი, რადგან უკვე გაქვთ Firebase ანგარიში |
| **Netlify** | უბრალოდ ფაილების drag-and-drop | კარგი ალტერნატივა |
| **GitHub Pages** | მოითხოვს Git-ის ცოდნას | თუ უკვე იცნობთ GitHub-ს |

### Firebase Hosting-ით ატვირთვის მოკლე გზა:
1. დააყენეთ Firebase CLI: `npm install -g firebase-tools`
2. ტერმინალში, საიტის საქაღალდეში: `firebase login` → `firebase init hosting`
3. აირჩიეთ თქვენი პროექტი, public დირექტორიად მიუთითეთ ის საქაღალდე სადაც ეს ფაილებია
4. `firebase deploy`

### Netlify-ით (უმარტივესი, კოდის გარეშე):
1. გახსენით [app.netlify.com/drop](https://app.netlify.com/drop)
2. გადაიტანეთ მთელი საქაღალდე (`index.html`, `admin.html`, `style.css`, `config.js`) ბრაუზერში
3. მიიღებთ დროებით ბმულს — შემდეგ ნაბიჯში დაუკავშირდება თქვენს დომენს

---

## ნაბიჯი 5 — nmtshop.ge დომენის დაკავშირება

1. შედით დომენის რეგისტრატორის პანელში (სადაც `nmtshop.ge` შეიძინეთ) → DNS Settings
2. აირჩეულმა ჰოსტინგმა (Firebase Hosting / Netlify) მოგცემთ ზუსტ ინსტრუქციას — ჩვეულებრივ:
   - **A ჩანაწერი**, რომელიც მიუთითებს ჰოსტინგის IP-ზე, ან
   - **CNAME ჩანაწერი**, რომელიც მიუთითებს ჰოსტინგის მისამართზე
3. დაამატეთ იგივე `nmtshop.ge`-ც და `www.nmtshop.ge`-ც თითოეული ჰოსტინგის პანელშივე (Custom domain settings), რათა SSL/HTTPS ავტომატურად გაიცეს.
4. დაბრუნდით Firebase Authentication-ში → Settings → Authorized domains → დაამატეთ `nmtshop.ge`, თორემ ადმინის შესვლა არ იმუშავებს თქვენს დომენზე.
5. დაელოდეთ რამდენიმე საათს (ზოგჯერ 24-48 სთ-მდე) DNS-ის გავრცელებას.

---

## რა არის სამომავლო გასაუმჯობესებელი (სურვილისამებრ)

- **კატეგორიები/ფილტრები** — თუ პროდუქცია იზრდება, გამოგადგებათ კატეგორიებით დაჯგუფება
- **ძებნის ველი** კატალოგის გვერდზე
- **მეორე ადმინის** დამატება — Authentication → Users-ში უბრალოდ დაამატეთ კიდევ ერთი მეილი

თუ რომელიმე ნაბიჯზე გაგიჭირდებათ ან გინდათ რომელიმე ფუნქციის დამატება — მომწერეთ, ერთად გავაგრძელებთ.
