# NOVA Android — GitHub থেকে APK ডাউনলোড

এটি NOVA-এর Android Studio project। বর্তমান ওয়েব UI রাখা হয়েছে। Android অংশ ভিডিওকে অ্যাপের নিজস্ব private storage-এ কপি করে এবং Android-এর system chooser দিয়ে ইনস্টল থাকা compatible player-এ খুলে দেয়। “Save to phone” বোতাম Android-এর Save dialog দিয়ে Downloads বা নির্বাচিত ফোল্ডারে কপি করতে পারে। VLC/MX Player দেখা যাবে যদি ফোনে ইনস্টল থাকে এবং Android ওই ফাইলের জন্য তাদের compatible হিসেবে চেনে।

## GitHub-এ আপলোড
1. GitHub-এ **New repository** তৈরি করো (Public) এবং ZIP Extract করো।
2. Extract করা `nova_android_github` ফোল্ডারের ভেতরের সব ফাইল—`.github` ফোল্ডারসহ—repository-র root-এ upload করো। `app`, `build.gradle`, `settings.gradle`, `gradle.properties`, `.github` root-এ থাকতে হবে।
3. Commit changes করো।
4. Repository → **Actions** → `Build NOVA Android APK` workflow খুলে run-এর ফলাফল দেখো। প্রথম push-এই workflow স্বয়ংক্রিয়ভাবে চলবে।
5. Build সফল হলে run-এর **Artifacts → NOVA-Android-APK** থেকে APK ডাউনলোড করা যাবে।
6. Release link পেতে আগে উপরের চারটি signing secret সেট করো, তারপর **Releases → Create a new release** থেকে `v1.0.0` tag publish করো। Secret ছাড়া version tag দিলে workflow ইচ্ছাকৃতভাবে বন্ধ হবে—debug APK-কে স্থায়ী Release হিসেবে প্রকাশ করবে না। পরের version-এ `v1.0.1`, `v1.0.2` ইত্যাদি tag ব্যবহার করো।

## Chrome দিয়ে ডাউনলোড
GitHub repository → **Releases** → সর্বশেষ release → APK asset → Download। Chrome download শেষ হলে APK খুলে Android-এর installation নির্দেশনা অনুসরণ করো। ফোনে প্রয়োজন হলে Settings-এ Chrome-কে এই উৎস থেকে app install করার অনুমতি দিতে হবে।

## ভবিষ্যৎ আপডেটের জন্য স্থায়ী signing key (প্রস্তাবিত)
একই signing key ব্যবহার না করলে নতুন APK পুরোনো APK-এর ওপর update হিসেবে install হবে না। প্রথম ব্যবহারেই স্থায়ী key সেট করো, তাহলে পরের version-এ NOVA-এর private video copy ও app data থাকবে।

1. কম্পিউটারে Java JDK-এর `keytool` দিয়ে একবার keystore তৈরি করো:
   `keytool -genkeypair -v -keystore nova-release.jks -alias nova -keyalg RSA -keysize 2048 -validity 10000`
2. PowerShell-এ keystore-টি Base64 করে clipboard-এ কপি করো:
   `[Convert]::ToBase64String([IO.File]::ReadAllBytes("nova-release.jks")) | Set-Clipboard`
3. GitHub repository → **Settings → Secrets and variables → Actions → New repository secret**-এ এই চারটি secret তৈরি করো: `NOVA_KEYSTORE_BASE64` (clipboard-এর Base64), `NOVA_STORE_PASSWORD`, `NOVA_KEY_ALIAS` (সাধারণত `nova`), `NOVA_KEY_PASSWORD`।
4. Keystore ফাইলটি কখনো repository-তে upload/commit করবে না। হারিয়ে গেলে একই signing key দিয়ে update বানানো যাবে না।

চারটি secret দেওয়া থাকলে workflow একই key-তে release-signed APK বানায় এবং প্রতিটি CI run-এ versionCode স্বয়ংক্রিয়ভাবে বাড়ে। Secret না থাকলে branch push / manual test run-এ শুধু debug-signed test APK তৈরি হয়। নিরাপত্তার জন্য version tag (`v1.0.0`) থেকে Release প্রকাশ করতে signing secrets বাধ্যতামূলক; secrets না থাকলে workflow থেমে যাবে, ভুল করে debug-signed APK Release-এ যাবে না। Debug APK-তে ভিডিও রাখার পরে uninstall করে অন্য signature-এর APK বসালে app-এর private video copy মুছে যাবে।

## গুরুত্বপূর্ণ সীমাবদ্ধতা ও পরীক্ষা
- UI assets APK-এর ভেতরে bundle করা; `WebViewAssetLoader` (AndroidX WebKit 1.15.0) এগুলোকে `https://appassets.androidplatform.net/` secure origin-এ পরিবেশন করে। Build setup-এ compile SDK 35, AGP 8.6.1, Gradle 8.7 ও Build Tools 34.0.0 মিলিয়ে pin করা আছে।
- Android file picker থেকে ভিডিও 256 KB buffer দিয়ে stream করে private storage-এ কপি হয়; ভিডিওকে Base64 বা একসঙ্গে RAM-এ load করা হয় না। প্রতিটি import-এর temporary `.part` ফাইল আলাদা নামে তৈরি হয়, তাই একই episode-এ দ্রুত পরপর import হলে ফাইল একে অন্যের সঙ্গে মিশে যায় না। নতুন কপি সম্পূর্ণ sync হওয়ার পর atomic rename হয়, তাই মাঝপথে ব্যর্থ হলে আগের কপি অক্ষত থাকে। Native bridge অনুপলব্ধ হলে WebView-এর file-input chooser fallback ব্যবহার হয়। Native video-র “Save to phone” অপশন Android Storage Access Framework দিয়ে নির্বাচিত URI-তে buffer ধরে export করে। কপি চলার সময় ফোনে যথেষ্ট ফাঁকা storage লাগবে।
- VLC/MX Player chooser Android-এর ACTION_VIEW intent ব্যবহার করে; উপযুক্ত player ইনস্টল থাকলে এবং সেটি ওই container/MIME গ্রহণ করলে তবেই তালিকায় দেখা যাবে। সব codec, resolution বা hardware decoding-এ শতভাগ playback নিশ্চয়তা দেওয়া যায় না।
- NOVA uninstall বা App data clear করলে private video copy মুছে যায়। Gallery থেকে মূল ভিডিও মুছলেও NOVA-এর copied file থাকবে, যতক্ষণ অ্যাপের data অক্ষত থাকে।
- Android SDK/Gradle build ও বাস্তব ফোনে test এই প্রস্তুতকরণ পরিবেশে চালানো যায়নি। GitHub Actions build সফল হওয়া এবং ফোনে VLC/MX Player পরীক্ষা এখনো আবশ্যক।
