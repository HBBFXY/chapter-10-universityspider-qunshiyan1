import requests
from bs4 import BeautifulSoup
import csv
import time

headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 Chrome/114.0.0.0 Safari/537.36'}

def main():
    all_uni = []  
    page = 1
    
    print("开始爬取软科大学排名...")
    
    while True:
        url = f'https://www.shanghairanking.cn/rankings/bcur/202411?page={page}'
        
        try:
            r = requests.get(url, headers=headers, timeout=10)
            r.encoding = 'utf-8'
            soup = BeautifulSoup(r.text, 'html.parser')
            table = soup.find('table', class_='rk-table')
            if not table:
                break 
            rows = table.find_all('tr')[1:]
            if not rows:
                break  
                
            for row in rows:
                cols = row.find_all('td')
                if len(cols) >= 10:
                    rank = cols[0].text.strip()
                    name = cols[1].text.strip()
                    province = cols[2].text.strip()
                    type_ = cols[3].text.strip()
                    score = cols[5].text.strip()
                    
                    all_uni.append({
                        '排名': rank,
                        '大学名称': name,
                        '省市': province,
                        '类型': type_,
                        '总分': score
                    })
            
            print(f"已爬取第{page}页，共{len(all_uni)}所大学")
            time.sleep(0.5)  # 简短延迟
            page += 1
            
        except Exception as e:
            print(f"第{page}页爬取失败: {e}")
            break
    if all_uni:
        with open('软科大学排名.csv', 'w', newline='', encoding='utf-8-sig') as f:
            writer = csv.DictWriter(f, fieldnames=['排名', '大学名称', '省市', '类型', '总分'])
            writer.writeheader()
            writer.writerows(all_uni)
        print(f"爬取完成！共{len(all_uni)}所大学数据已保存到'软科大学排名.csv'")
    else:
        print("未获取到任何数据")

if __name__ == "__main__":
    main()
