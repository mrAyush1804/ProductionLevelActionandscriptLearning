Dependency Guard Kya Karta Hai?
Dependency Guard ek Gradle Plugin hai jo Android Project ke dependencies ka snapshot store karta hai aur ensure karta hai ki koi bhi dependency bina track kiye change na ho.

Use Case:
Man lo tumhare Android project me kuch dependencies define hain aur tum chahti ho ki koi bhi unexpected dependency change na ho. Agar koi nai dependency add hoti hai ya update hoti hai, to GitHub Actions me build fail ho jayega taaki tum manually check kar sako.

Step-by-Step Process to Apply in Android
1️⃣ Dependency Guard Plugin Add Karo
Sabse pehle build.gradle.kts (Project-level) file me yeh plugin add karo:

kotlin
Copy
Edit
plugins {
    id("com.dropbox.dependency-guard") version "0.2.0"
}
Is plugin ka kaam dependencies ka baseline create karna aur future me changes track karna hai.

2️⃣ Baseline Generate Karo (First Time)
Jab tum first time Dependency Guard setup karoge, to ek baseline file create karni padegi jo tumhare project ki current dependencies ko store karegi.

👇 Is command ko run karo:

sh
Copy
Edit
./gradlew dependencyGuardBaseline
🔹 Yeh kya karega?

Current dependencies ka ek record generate karega.

Yeh file Git me commit hogi aur future builds ke comparison ke liye use hogi.

✅ Important: Is file ko Git me commit kar do, taki future me koi dependency change ho to detect ho sake.

3️⃣ Verify Dependencies with Dependency Guard
Agar tum check karna chahti ho ki dependencies change hui ya nahi, to yeh command run karo:

sh
Copy
Edit
./gradlew dependencyGuard
🔹 Yeh kya karega?

Jo baseline file store ki gayi thi, usko current dependencies se compare karega.

Agar koi bhi dependency change hoti hai, to build fail ho jayega.

🔴 Example: Agar tum build.gradle.kts me koi nai dependency add karti ho, jaise:

kotlin
Copy
Edit
dependencies {
    implementation("androidx.lifecycle:lifecycle-runtime-ktx:2.7.0")
}
Aur tum ./gradlew dependencyGuard run karti ho, to error aayega:

yaml
Copy
Edit
Dependency changed: androidx.lifecycle:lifecycle-runtime-ktx:2.6.2 -> 2.7.0
FAILURE: Build failed
Matlab, Dependency Guard bata raha hai ki dependency change ho gayi hai jo baseline se match nahi karti.

4️⃣ GitHub Actions Me Automation (CI/CD Setup)
GitHub Actions me Dependency Guard kaise apply karein?
👇 Tum .github/workflows/android-ci.yml file me yeh steps add karo:

yaml
Copy
Edit
name: Android CI

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  dependency_check:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Setup JDK 17
        uses: actions/setup-java@v3
        with:
          distribution: 'zulu'
          java-version: '17'

      - name: Grant Execute Permissions for Gradle
        run: chmod +x gradlew

      - name: Setup Gradle Cache
        uses: gradle/gradle-build-action@v2

      - name: Run Dependency Verification
        run: ./gradlew dependencyGuard

      - name: Auto-Fix Dependency Mismatch (For PRs)
        if: failure() && github.event_name == 'pull_request'
        run: |
          echo "Dependencies changed! Updating baseline..."
          ./gradlew dependencyGuardBaseline
          git config --global user.name 'github-actions'
          git config --global user.email 'github-actions@github.com'
          git add .
          git commit -m "Update dependency guard baseline"
          git push
5️⃣ Yeh GitHub Actions Me Kaise Kaam Karega?
Agar koi developer ek PR banata hai jo dependencies change karta hai, to yeh workflow check karega ki dependencies match karti hain ya nahi.

📌 Process:

Repository checkout karega.

Java JDK setup karega.

Gradle permission set karega.

Gradle caching karega taaki build fast ho.

Dependency Guard verify karega.

Agar dependencies match nahi hui, to build fail hoga.

Agar PR me dependency change hui hai, to auto-baseline update hoke push ho jayega.

Final Summary
✅ Dependency Guard Android project ke dependencies track karne aur unexpected dependency changes detect karne ke liye use hota hai.
✅ GitHub Actions ke saath integrate karke, tum ensure kar sakti ho ki koi bhi dependency bina track kiye update na ho.
✅ Agar koi dependency change hoti hai, to ya to build fail ho jayega ya PR ke andar baseline auto-update ho jayega. 🚀







---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
📸 Run Screenshot Tests using Roborazzi in Android
🎯 ./gradlew verifyRoborazziDebug Kya Karta Hai?
Yeh Roborazzi ka ek command hai jo screenshot-based testing ke liye use hota hai.
Iska kaam app ke UI screenshots capture karna aur unko expected output ke sath compare karna hota hai.

🔥 Roborazzi Setup Android Project Me Kaise Karein?
Step 1️⃣: Dependencies Add Karo
build.gradle (Module: app) me dependencies add karo:

gradle
Copy
Edit
dependencies {
    androidTestImplementation "io.github.takahirom.roborazzi:roborazzi:1.6.1"
}
Step 2️⃣: Gradle Plugin Enable Karo
gradle.properties me roborazzi enable karna hoga:

properties
Copy
Edit
android.experimental.testOptions.roborazzi=true
Step 3️⃣: Screenshot Test Likho
Android Instrumented Test likhna hoga jo screenshots capture karega.

kotlin
Copy
Edit
@RunWith(AndroidJUnit4::class)
class ScreenshotTest {
    @get:Rule
    val composeTestRule = createAndroidComposeRule<MainActivity>()

    @Test
    fun captureHomeScreen() {
        composeTestRule.onRoot().captureRoboImage()
    }
}
Ye test Home Screen ka screenshot lega aur compare karega.

⚙️ GitHub Actions Me Roborazzi Setup Kaise Karein?
Agar tum GitHub Actions me automate karna chahte ho, to ye workflow file add karo:

yaml
Copy
Edit
name: Run Screenshot Tests

on:
  push:
    branches:
      - main

jobs:
  screenshotTest:
    runs-on: macos-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3

      - name: Set up Java
        uses: actions/setup-java@v3
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: Install Android SDK
        uses: android-actions/setup-android@v3

      - name: Run Roborazzi Screenshot Tests
        run: ./gradlew verifyRoborazziDebug
💡 Roborazzi Ka Best Use Case Kya Hai?
✅ UI Regression Testing → UI changes track karne ke liye
✅ Dark Mode / Light Mode Test → Dono themes ke screenshots compare karna
✅ Different Screen Sizes Test → UI responsiveness check karna
✅ Animations & Transitions Debugging

🚀 Conclusion
🔹 Roborazzi screenshot testing ke liye use hota hai.
🔹 verifyRoborazziDebug UI screenshots capture karke compare karta hai.
🔹 GitHub Actions me automation ke liye MacOS aur Android SDK setup karna padta hai.
🔹 Yeh visual regression testing ke liye best hai! 🚀













------------------------------------------------------------------------------------------------------------------------------------------------------------
🚀 Automated Screenshot Testing using GitHub Actions
Agar tum chaahte ho ki har pull request (PR) ya commit pe automatically screenshot test ho jaye, to GitHub Actions ka use kar sakte ho.
Isme tests run honge, screenshots generate honge, aur changes verify honge.

✅ 1️⃣ Process Overview
🔹 Step 1: GitHub Actions setup karein (.github/workflows/screenshot-tests.yml)
🔹 Step 2: Emulator start karein aur tests run karein
🔹 Step 3: Screenshots ko verify karein aur GitHub pe upload karein
🔹 Step 4: Agar screenshot change ho gaye, to PR fail ho jaye

🛠 2️⃣ Setup GitHub Actions Workflow
👉 GitHub me ek naye workflow file banao:
📌 Path: .github/workflows/screenshot-tests.yml

yaml
Copy
Edit
name: 📸 Screenshot Testing

on:
  pull_request:
    branches:
      - main  # Jab bhi PR aayega "main" branch pe, yeh test run hoga

jobs:
  screenshot-tests:
    runs-on: macos-latest  # Android emulator ke liye MacOS chahiye

    steps:
      - name: 📥 Checkout Repository
        uses: actions/checkout@v4

      - name: 📦 Setup JDK & Gradle
        uses: actions/setup-java@v8
        with:
          distribution: 'temurin'
          java-version: '17'  # Android latest versions ke liye Java 17 best hai

      - name: 🏗 Setup Android SDK
        uses: android-actions/setup-android@v3

      - name: 📱 Start Android Emulator
        uses: reactivecircus/android-emulator-runner@v2
        with:
          api-level: 30
          arch: x86_64
          script: ./gradlew connectedDebugAndroidTest  # Emulator par test run karega

      - name: 📸 Run Screenshot Tests (Roborazzi)
        run: ./gradlew verifyRoborazziDebug

      - name: 📝 Upload Screenshots as Artifacts
        uses: actions/upload-artifact@v4
        with:
          name: screenshots
          path: app/build/outputs/roborazzi  # Yahan screenshots save hote hain
🧐 3️⃣ Yeh Workflow Kya Kar Raha Hai?
Step	Kaam
📥 Checkout Repository	GitHub Actions me repo download hota hai
📦 Setup JDK & Gradle	Java aur Gradle ko setup karta hai
🏗 Setup Android SDK	Android SDK setup hota hai
📱 Start Android Emulator	Emulator ko start karta hai
📸 Run Screenshot Tests	./gradlew verifyRoborazziDebug se screenshots verify hote hain
📝 Upload Screenshots	Artifacts ke through screenshots GitHub pe upload hote hain
🚀 4️⃣ Verify Changes in Screenshots
Agar koi screenshot change hota hai, to PR me "checks failed" dikhayega.
Iska matlab: ✅ Agar UI same hai, to PR pass hoga.
❌ Agar UI change ho gayi, to PR fail hoga aur review karna padega.

⚡ 5️⃣ Screenshot Diffing (Changes Track Karna)
Agar tum screenshot changes track karna chahte ho, to GitHub pe screenshots upload hone ke baad compare kar sakte ho.

🔹 Yeh command use karke local system me changes compare karo:

sh
Copy
Edit
./gradlew recordRoborazziDebug
🔹 Fir naye screenshots ko GitHub pe commit karo:

sh
Copy
Edit
git add app/src/main/roborazzi
git commit -m "Updated screenshots"
git push origin <branch-name>
🔄 6️⃣ Agar Changes Accept Karne Hain
Agar tumhe screenshots update karne hain, to: 1️⃣ Local system me screenshots regenerate karo:

sh
Copy
Edit
./gradlew recordRoborazziDebug
2️⃣ Changes commit karke PR me push karo:

sh
Copy
Edit
git add app/src/main/roborazzi
git commit -m "Updated UI screenshots"
git push origin <branch-name>
🔥 Final Summary
1️⃣ GitHub Actions ko setup karo (screenshot-tests.yml)
2️⃣ Har PR pe screenshots generate aur verify honge
3️⃣ Agar UI change hoti hai, to PR fail ho jayega
4️⃣ Agar naye screenshots sahi hain, to manually accept karke commit karo

🎯 Ab tumhare app me har UI change automatic verify ho jayega! 🚀









