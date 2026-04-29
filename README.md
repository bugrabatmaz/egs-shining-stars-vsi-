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

SELAM DÜNYA

import tkinter as tk
import math
import json
import time
from datetime import datetime

class VSISimulator:
    def __init__(self, root):
        self.root = root
        self.root.title("VSI Simülatörü")
        self.root.geometry("800x800")
        
        self.target_fpm = 0.0
        self.current_fpm = 0.0
        self.smoothing_factor = 0.05
        
        self.is_metric = False
        self.is_failed = False
        
        self.logs = []
        self.last_log_time = time.time()
        
        self.setup_ui()
        self.update_instrument()
        
# Animasyon döngüsünü başlat

    def setup_ui(self):
        self.canvas = tk.Canvas(self.root, width=400, height=400, bg="#1e1e1e", highlightthickness=0)
        self.canvas.pack(pady=20)
        
        self.draw_static_dial()
       
        self.needle_id = self.canvas.create_polygon(0, 0, 0, 0, fill="#ffffff", outline="#aaaaaa")
        self.center_pin_id = self.canvas.create_oval(188, 188, 212, 212, fill="#2c3e50", outline="#7f8c8d", width=2)
        
# Arıza (OFF) Bayrağı (Başlangıçta gizli)

        self.off_flag_bg = self.canvas.create_rectangle(220, 190, 270, 220, fill="red", outline="white", state="hidden")
        self.off_flag_text = self.canvas.create_text(245, 205, text="OFF", fill="white", font=("Arial", 12, "bold"), state="hidden")
        
 # Dinamik Birim ve Değer Metni
        
        self.value_text_id = self.canvas.create_text(200, 270, text="0 FPM", fill="#f39c12", font=("Courier", 16, "bold"))

  # Alarm Çerçevesi 
        
        self.warning_label = tk.Label(self.root, text="", font=("Arial", 14, "bold"), fg="red")
        self.warning_label.pack()

 # 2. KONTROLLER
 
        control_frame = tk.Frame(self.root)
        control_frame.pack(fill="x", padx=50)

        tk.Label(control_frame, text="Dikey Hız (Hedef):").pack()
        self.fpm_slider = tk.Scale(control_frame, from_=2000, to=-2000, orient="horizontal", 
                                   length=400, command=self.on_slider_change)
        self.fpm_slider.pack(pady=10)

  # Butonlar Çerçevesi
  
        btn_frame = tk.Frame(self.root)
        btn_frame.pack(pady=20)

        self.btn_unit = tk.Button(btn_frame, text="Birim: FPM", width=15, command=self.toggle_unit)
        self.btn_unit.grid(row=0, column=0, padx=10)

        self.btn_fail = tk.Button(btn_frame, text="Sistem: NORMAL", width=15, command=self.toggle_failure, bg="green", fg="white")
        self.btn_fail.grid(row=0, column=1, padx=10)

        self.btn_log = tk.Button(btn_frame, text="Logları İndir (JSON)", width=15, command=self.export_logs)
        self.btn_log.grid(row=0, column=2, padx=10)

 # --- ETKİLEŞİM FONKSİYONLARI ---
 
    def on_slider_change(self, val):
        if not self.is_failed:
            self.target_fpm = float(val)

    def toggle_unit(self):
        self.is_metric = not self.is_metric
        self.btn_unit.config(text="Birim: m/s" if self.is_metric else "Birim: FPM")

    def toggle_failure(self):
        self.is_failed = not self.is_failed
        if self.is_failed:
            self.btn_fail.config(text="Sistem: ARIZA (OFF)", bg="red")
            self.target_fpm = 0 # Arıza anında hedef hız sıfıra düşer
            self.fpm_slider.set(0)
        else:
            self.btn_fail.config(text="Sistem: NORMAL", bg="green")

    def export_logs(self):
        filename = f"vsi_logs_{datetime.now().strftime('%Y%m%d_%H%M%S')}.json"
        with open(filename, 'w', encoding='utf-8') as f:
            json.dump(self.logs, f, indent=4)
        print(f"Loglar başarıyla kaydedildi: {filename}")


    def update_instrument(self):
        # İnterpolasyon (Lerp) - Kapsül gecikmesi simülasyonu
        self.current_fpm += (self.target_fpm - self.current_fpm) * self.smoothing_factor

 # Uyarı Sistemi
        
        if abs(self.current_fpm) > 1500:
            self.warning_label.config(text="DİKKAT: YÜKSEK DİKEY HIZ")
            self.canvas.config(bg="#c0392b")
        else:
            self.warning_label.config(text="")
            self.canvas.config(bg="#2c3e50")

  # Veri Loglama (Saniyede 1 kez)
  
        current_time = time.time()
        if current_time - self.last_log_time >= 1.0:
            self.logs.append({
                "timestamp": datetime.now().isoformat(),
                "target_fpm": round(self.target_fpm, 2),
                "current_fpm": round(self.current_fpm, 2),
                "status": "FAILED" if self.is_failed else "OK"
            })
            self.last_log_time = current_time

  # Ekranı Yeniden Çiz
  
        self.draw_vsi()

 # Döngüyü 16ms sonra (yaklaşık 60 FPS) tekrar çağır
 
        self.root.after(16, self.update_instrument)

    def draw_static_dial(self):
        cx, cy, r = 200, 200, 170
        
  # Dış Çerçeve (Bezel)
  
        self.canvas.create_oval(cx-r-10, cy-r-10, cx+r+10, cy+r+10, fill="#111111", outline="#555555", width=6)
        self.canvas.create_oval(cx-r, cy-r, cx+r, cy+r, fill="#1e1e1e", outline="#333333", width=2)

  # Bilgi Metinleri
  
        self.canvas.create_text(cx, cy-60, text="VERTICAL SPEED", fill="#888888", font=("Arial", 10, "bold"))
        self.canvas.create_text(cx, cy+60, text="100 FEET PER MINUTE", fill="#888888", font=("Arial", 8))
        self.canvas.create_text(cx-60, cy-50, text="UP", fill="white", font=("Arial", 14, "bold"))
        self.canvas.create_text(cx-60, cy+50, text="DN", fill="white", font=("Arial", 14, "bold"))

 # Çentikler (Ticks) ve Sayılar
 
        for fpm in range(-2000, 2001, 100):
            ratio = fpm / 2000.0
            angle_deg = 180 + (ratio * 135)
            angle_rad = math.radians(angle_deg)

            is_major = (fpm % 500 == 0) # 500, 1000, 1500, 2000 noktaları
            
 # Çizgi uzunlukları
 
            outer_r = r - 5
            inner_r = r - 25 if is_major else r - 12
            
            x1 = cx + inner_r * math.cos(angle_rad)
            y1 = cy + inner_r * math.sin(angle_rad)
            x2 = cx + outer_r * math.cos(angle_rad)
            y2 = cy + outer_r * math.sin(angle_rad)

            line_width = 4 if is_major else 2
            self.canvas.create_line(x1, y1, x2, y2, fill="white", width=line_width)

 # Sayıları Yazdırma (Sadece ana çentiklere ve 0 haricindekilere)
 
            if is_major:
                val = abs(fpm) // 100 # 500 yerine 5, 1000 yerine 10 yazar
                text_r = r - 45
                tx = cx + text_r * math.cos(angle_rad)
                ty = cy + text_r * math.sin(angle_rad)
                
  # 0 noktasını biraz daha dışarı ve belirgin yazalım
  
                if fpm == 0:
                    self.canvas.create_text(cx - r + 35, cy, text="0", fill="white", font=("Arial", 20, "bold"))
                else:
                    self.canvas.create_text(tx, ty, text=str(val), fill="white", font=("Arial", 16, "bold"))
    
    def draw_vsi(self):
        cx, cy, r = 200, 200, 170

 # Değer ve Birim Metni Güncellemesi
 
        display_val = self.current_fpm
        unit_text = "FPM"
        if self.is_metric:
            display_val = self.current_fpm * 0.00508
            unit_text = "m/s"
            
        self.canvas.itemconfig(self.value_text_id, text=f"{round(display_val, 1)} {unit_text}")
 # Arıza (OFF) Bayrağı Görünürlüğü
 
        flag_state = "normal" if self.is_failed else "hidden"
        self.canvas.itemconfig(self.off_flag_bg, state=flag_state)
        self.canvas.itemconfig(self.off_flag_text, state=flag_state)

   # İbre (Needle) Açısını Hesaplama
   
        clamped_fpm = max(-2000, min(2000, self.current_fpm))
        ratio = clamped_fpm / 2000.0
        angle_deg = 180 + (ratio * 135) 
        angle_rad = math.radians(angle_deg)
        
# Gerçekçi İbre Geometrisi (Poligon Köşeleri)
 # İbrenin ucu, tabanın sol ve sağ genişliği, ve arka kuyruk
 
        tip_x = cx + (r - 10) * math.cos(angle_rad)
        tip_y = cy + (r - 10) * math.sin(angle_rad)
        
 # Tabanı genişletmek için açıya 90 derece (Pi/2) ekleyip çıkarıyoruz
 
        base_width = 8
        left_x = cx + base_width * math.cos(angle_rad - math.pi/2)
        left_y = cy + base_width * math.sin(angle_rad - math.pi/2)
        
        right_x = cx + base_width * math.cos(angle_rad + math.pi/2)
        right_y = cy + base_width * math.sin(angle_rad + math.pi/2)
        
        tail_x = cx - 30 * math.cos(angle_rad) # Kuyruk uzantısı
        tail_y = cy - 30 * math.sin(angle_rad)

  # Poligonu yeni koordinatlarla güncelle
        self.canvas.coords(self.needle_id, tip_x, tip_y, left_x, left_y, tail_x, tail_y, right_x, right_y)
        

        self.canvas.tag_raise(self.center_pin_id)

if __name__ == "__main__":
    root = tk.Tk()
    app = VSISimulator(root)
    root.mainloop()
    
