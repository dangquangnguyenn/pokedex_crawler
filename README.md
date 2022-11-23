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
### 1. Lấy danh sách các id riêng biệt của từng Pokemon
```
pokemon_id = soup.findAll('span', {'class': 'infocard-cell-data'})
```
### 2. Danh sách các url của từng Pokemon
- Chúng ta có thể dùng id hoặc tên của Pokemon đó để làm subdirectory.
Ví dụ: Pokemon có tên là Bulbasaur, id là 001:

https://pokemondb.net/pokedex/bulbasaur

https://pokemondb.net/pokedex/001 hay https://pokemondb.net/pokedex/1
```
urls.append('https://pokemondb.net/pokedex/' + str(id[i]))
```
### 3. Thu thập dữ liệu
#### 3.1. Thông tin của một pokemon
- Lưu ý: ID của Pokemon sẽ được thêm vào sau cùng sau đó đặt ID làm cột index, ở đây chúng ta chỉ xét các thuộc tính như: tên, hệ, loài,...
- Sẽ có thêm các cơ sở dữ liệu riêng để mô tả chi tiết các đặc tính cũng như đặc tính ẩn và tương khắc hệ
- Các thuộc tính của một Pokemon sẽ bao gồm:

            - name: Tên

            - types_list: Hệ của Pokemon, mỗi con sẽ mang 1 hệ hoặc song hệ

            - species: Loài

            - height: Chiều cao

            - weight: Cân nặng

            - ability: Đặc tính

            - hid_ability (hidden ability): Đặc tính ẩn

            - catch_rate: Tỉ lệ bắt trúng bằng Poke Ball, bị ảnh hưởng bởi rất nhiều yếu tố:
                    * HP hiện tại
                    * Loại bóng (Poke Ball, Great Ball, Ultra Ball,...), Master Ball có tỷ lệ thu phục là 100% với tất cả Pokemon
                    * Trạng thái hiện tại (Choáng, phỏng, nhiễm độc, ngủ, tê liệt,...)
        
            - base_fs (base friendship): Chỉ số thân thiện cơ bản, một số Pokemon có chỉ số thân thiện là 0, bọn này có vẻ không ưa con người cho lắm

            - base_exp: Số lượng exp cần thiết để lên 1 level bất kỳ được tính theo hàm mũ. Mỗi loại có công thức khác nhau và khá phức tạp:
                    * Ví dụ 1 công thức đơn giản nhất cho mọi người:
                        Loại 1 000 000: exp = n^3 
                        n là level cần lên.
                        -> Để lên lv3 Pokemon loại này cần tổng cộng là 27exp trừ số exp hiện có sẽ ra số exp cần phải luyện thêm
            
            - growth_rate: Tốc độ tăng trưởng:
                    * Erratic: 600.000 exp ở level 100
                    * Fast: 800.000 điểm kinh nghiệm ở cấp 100
                    * Medium Fast: 1.000.000 điểm kinh nghiệm ở cấp 100
                    * Medium Slow: 1.059.860 điểm kinh nghiệm ở cấp 100
                    * Slow: 1.250.000 điểm kinh nghiệm ở cấp 100
                    * Fluctuating: 1.640.000 exp ở cấp 100:
        
            - genders: Giới tính theo tỷ lệ

            - Base Stats: Chỉ số cơ bản, bao gồm: 
                    * hp (Health Point): Sinh mệnh
                    * atk (Attack): Tấn công
                    * defn (Defense): Phòng thủ, giáp (Vì def là khai báo hàm nên không dùng để khai báo biến cho Defense được :v)
                    * sp_atk (Special Attack): Tấn công đặc biệt, sát thương phép
                    * sp_def (Special Defense): Phòng thủ đặc biệt, kháng phép
                    * spd (Speed): Tốc độ (tương tự như các game xoay tua, Pokemon tốc độ cao hơn sẽ được đánh trước)
                    
            - total: Tổng base stats
- Hàm get_pokemon_info(soup) giúp lấy ra thông tin của một Pokemon có ID chỉ định. Sau đó sẽ dùng vòng lặp để lấy thông tin toàn bộ Pokemon trong Pokedex
```
def get_pokemon_info(soup):
  # YOUR CODE HERE
  ...
  return name, types_list, species, height, weight, ability, hid_ability, catch_rate, base_fs, base_exp, growth_rate, genders, hp, atk, defn, sp_atk, sp_def, spd, total 
```
#### 3.2. Tương khắc hệ
- Tương khắc hệ (Type Defenses) của một Pokemon là khả năng chịu đòn của một Pokemon trước những đòn tấn công mang hệ khác. Các Pokemon Trainer sẽ dựa vào tương khắc hệ này của chúng nhằm chọn ra Pokemon có ưu thế về hệ (Type Advantages) tốt nhất để đối đầu với Pokemon của đối thủ.

        Ví dụ: Bulbasaur mang song hệ là Grass/Poison:
                - Fire -> Grass/Poison = 2 <=> Đòn đánh hệ Fire sẽ gây gấp đôi sát thương lên Pokemon này
                - Grass -> Grass/Poison = 1/4   <=> Đòn đánh hệ Grass sẽ gây 0.25 sát thương lên Pokemon này

- Hàm Type_Defenses(soup) sẽ trả về tương khắc hệ của Pokemon có url tương ứng. 
```
def type_defense(soup):
  # YOUR CODE HERE
  ...
  return type_defenses
```
#### 3.3. Đặc tính
- Đặc tính (Ability) và đặc tính ẩn (Hidden Ability)
- Hàm get_abilities_description(soup) mô tả chi tiết về các đặc tính và đặc tính ẩn của từng Pokemon
```
def get_abilities_description(soup):
  # YOUR CODE HERE
  ...
  return ab_des, hid_ab_des
```
#### 3.4. Bắt đầu lấy thông tin, tương khắc hệ, đặc tính của từng Pokemon
```
data = []           # Thông tin
type_defenses = []  # Tương khắc hệ
abilities = []      # Bao gồm đặc tính và đặc tính ẩn
```











