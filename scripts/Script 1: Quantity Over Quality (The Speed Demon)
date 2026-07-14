import base64
import os
import time
import random
from playwright.sync_api import sync_playwright

def encode_number_to_base64(num_str):
    bytes_data = num_str.encode('utf-8')
    base64_bytes = base64.b64encode(bytes_data)
    return base64_bytes.decode('utf-8')

def download_and_render_pdfs(start, end):
    output_dir = "rendered_player_copies"
    if not os.path.exists(output_dir):
        os.makedirs(output_dir)

    with sync_playwright() as p:
        print("\n[+] Initializing automated printing engine...")
        
        browser = p.chromium.launch(
            headless=True,
            executable_path="/usr/bin/chromium",
            args=["--no-sandbox", "--disable-setuid-sandbox", "--disable-gpu"]
        )
        
        context = browser.new_context(
            viewport={"width": 1280, "height": 800},
            user_agent="Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"
        )
        page = context.new_page()
        page.set_default_timeout(60000) 

        for i in range(start, end + 1):
            padded_number = f"{i:06d}"
            base64_id = encode_number_to_base64(padded_number)
            
            url = f"https://[Redacted]/generate_pdf.php?id={base64_id}"
            output_file_path = os.path.join(output_dir, f"{padded_number}.pdf")
            
            print(f" -> Processing Record #{padded_number}...")
            
            max_retries = 3
            success = False
            
            for attempt in range(max_retries):
                try:
                    response = page.goto(url, wait_until="domcontentloaded")
                    
                    if response and response.status == 200:
                        # 1. Wait for dynamic layout elements to fully load
                        time.sleep(3.5) 
                        
                        # 2. Inject CSS layout fixes to prevent two-page overflow spikes
                        page.add_style_tag(content="""
                            @media print {
                                html, body {
                                    height: 99% !important;
                                    font-size: 11px !important;
                                    margin: 0 !important;
                                    padding: 0 !important;
                                    page-break-inside: avoid !important;
                                    -webkit-print-color-adjust: exact !important;
                                }
                                .wrapper, .container, table {
                                    page-break-inside: avoid !important;
                                    break-inside: avoid !important;
                                    margin-bottom: 2mm !important;
                                }
                                img, .photo-box {
                                    max-height: 120px !important;
                                }
                            }
                        """)
                        
                        # 3. Print with layout metrics constrained to a single sheet
                        page.pdf(
                            path=output_file_path, 
                            format="A4",
                            print_background=True,
                            prefer_css_page_size=False,
                            margin={"top": "6mm", "bottom": "6mm", "left": "8mm", "right": "8mm"}
                        )
                        print(f"    [✔] Successfully saved uniform PDF: {padded_number}.pdf")
                        success = True
                        break
                    else:
                        status = response.status if response else "No Response"
                        print(f"    [!] Attempt {attempt+1}: Server returned status {status}")
                
                except Exception as e:
                    print(f"    [!] Attempt {attempt+1} failed connection: {str(e).splitlines()[0]}")
                
                time.sleep(2)
            
            if not success:
                print(f"    [❌] Permanent failure for Record #{padded_number} after {max_retries} attempts.")
            
            time.sleep(random.uniform(1.5, 3.5))

        browser.close()
    print(f"\n[+] Task finished! Check your folder: '{output_dir}/'")

if __name__ == "__main__":
    print("=" * 50)
    print("     AUTOMATED HTML-TO-PDF PRINTER (NO MANUAL CTRL+P)     ")
    print("=" * 50)
    
    try:
        start_input = int(input("Enter STARTING record number (e.g., 100001): "))
        end_input = int(input("Enter ENDING record number (e.g., 100005): "))
        
        if start_input > end_input:
            print("[!] Error: Starting number cannot be higher than ending number.")
        else:
            download_and_render_pdfs(start_input, end_input)
    except ValueError:
        print("[!] Error: Inputs must be numerical values only.")
