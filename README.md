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
