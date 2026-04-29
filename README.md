# Shining Stars* VSI
**Vertical Speed Indicator Simulator** 

# Özellikler

-Göstergelerin gerçek zamanlı animasyonu

-Kullanıcı girişi ile değer değiştirme 

-Gerçek enstrümana benzer görsel tasarım

-Birim dönüşümü (feet -> metre)

-Kritik değerlerde görsel veya sesli alarm/uyarı

# KODLAR

KODLAR

CART

CURT

FALAN

YAZI DENEME

#include <Wire.h>
#include <Adafruit_BMP280.h>

Adafruit_BMP280 bmp;

float prevAlt = 0;
unsigned long prevTime = 0;

void setup() {
  Serial.begin(9600);
  bmp.begin(0x76);

  prevAlt = bmp.readAltitude(1013.25);
  prevTime = millis();
}

void loop() {
  float currentAlt = bmp.readAltitude(1013.25);
  unsigned long currentTime = millis();

  float dt = (currentTime - prevTime) / 1000.0; // saniye
  float vsi = (currentAlt - prevAlt) / dt;      // m/s
  float vsi_fpm = vsi * 196.85;                 // ft/min

  Serial.print("VSI: ");
  Serial.print(vsi_fpm);
  Serial.println(" ft/min");

  prevAlt = currentAlt;
  prevTime = currentTime;

  delay(1000);
}



SELAM DÜNYA
