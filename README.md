name: Compilar OkPiso APK Limpia

on:
  push:
    branches: [ main, master ]
  workflow_dispatch:

jobs:
  build-android-apk:
    runs-on: ubuntu-latest

    steps:
      - name: 1. Descargar archivos del repositorio
        uses: actions/checkout@v4

      - name: 2. Preparar Node.js y Java 17
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: 3. Configurar Java JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: 4. Configurar Android SDK
        uses: android-actions/setup-android@v3

      - name: 5. Preparar carpeta web (www) con index.html
        run: |
          mkdir -p www
          cp index.html www/index.html

      - name: 6. Inicializar Capacitor y generar proyecto Android nativo
        run: |
          npm init -y
          npm install @capacitor/core @capacitor/cli @capacitor/android
          npx cap init "OkPiso" "com.sitorm.okpiso" --web-dir www
          npx cap add android

      - name: 7. Aplicar icono personalizado okpiso-icon.png si existe
        run: |
          if [ -f "okpiso-icon.png" ]; then
            for dir in android/app/src/main/res/mipmap-*; do
              cp okpiso-icon.png "$dir/ic_launcher.png"
              cp okpiso-icon.png "$dir/ic_launcher_round.png"
              cp okpiso-icon.png "$dir/ic_launcher_foreground.png"
            done
          fi

      - name: 8. Compilar APK limpia y sin anuncios
        run: |
          cd android
          chmod +x gradlew
          ./gradlew assembleDebug
          cp app/build/outputs/apk/debug/app-debug.apk ../OkPiso.apk

      - name: 9. Publicar OkPiso.apk para descargar
        uses: actions/upload-artifact@v4
        with:
          name: OkPiso-APK-Limpia
          path: OkPiso.apk
