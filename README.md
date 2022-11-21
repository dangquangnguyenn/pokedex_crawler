# pokedex_crawler
## Giới thiệu sơ lược
- Có thể bạn chưa biết, Pocket Monsters viết tắt là Pokemon, một series game cũng như anime đình đám, series game Pokemon phát triển bởi GameFreak hiện vẫn đang làm mưa làm gió trên các hệ máy Nitendo và trên thị trường game toàn cầu. Nhân dịp gen9 (Scarlet & Violet) của game vừa ra mắt, chúng ta sẽ điểm lại các pokemon từ những thế hệ trước và thông số cũng như đặc tính của chúng thông qua Pokedex. Ở đây, chúng ta sẽ sử dụng "https://pokemondb.net/pokedex/all" là trang web chứa dữ liệu Pokedex, hay nói cách khác là danh sách của toàn bộ Pokemon ĐÃ xuất hiện xuyên suốt series game này. Let's go!

## Install và import
- bs4 (Beautiful Soup): Một thư viện Python dùng để phân tích các tài liệu HTML và XML. Nó tạo ra một cây phân tích cú pháp cho các trang được phân tích cú pháp có thể được sử dụng để trích xuất dữ liệu từ HTML, rất hữu ích cho web scrapping.
- requests: Cho phép gửi các yêu cầu HTTP bằng Python.
```
!pip install bs4
!pip install requests
```
import thêm một số thư viện:
```
import bs4
import requests
import pandas as pd
```

## Thu thập và xử lý dữ liệu
- Lưu ý: Có thể các Pokemon có rất nhiều dạng (vùng miền, mega evolution, mega X/mega Y, nguyên thủy (primal form), tấn công/phòng thủ,...), hình dạng Pokemon thay đổi nhưng id thì vẫn giữ nguyên, vì vậy sẽ xuất hiện trường hợp một id lặp lại nhiều lần, chúng ta chỉ xét hình dạng cơ bản của chúng nên chỉ cần lấy 1 id cho mỗi Pokemon thôi.
### 1. Lấy danh sách id riêng biệt cho từng pokemon
```
url = 'https://pokemondb.net/pokedex/all'
source = requests.get(url).text
soup = bs4.BeautifulSoup(source,'html.parser')

# Thu thập id của toàn bộ pokemon
pokemon_id = soup.findAll('span', {'class': 'infocard-cell-data'})
```




















