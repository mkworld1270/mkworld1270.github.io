# c# Enum 사용 시 주의사항

최근에 테스트를 하다보니 enum 데이터 결과가 이상하게 나오는 것을 발견

왜 이상하게 나오는지 체크해보았는데 Enum.TryParse의 동작이 내 예상과 다르게 동작


### 소스코드
```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text.RegularExpressions;

namespace Rextester
{
    public class Program
    {
        enum Team
        {
            NONE,
            TEAM1,
            TEAM2,
            TEAM3,
        }
         
        public static void Main(string[] args)
        {
            Team t;
            bool result;           
            
            result = Enum.TryParse("TEAM1", out t);            
            Console.WriteLine($"Result : {result}, enum : {t.ToString()}");
            
            result = Enum.TryParse("NOT EXIST", out t);            
            Console.WriteLine($"Result : {result}, enum : {t.ToString()}");
            
            result = Enum.TryParse("2", out t);            
            Console.WriteLine($"Result : {result}, enum : {t.ToString()}");
                        
            result = Enum.TryParse("5", out t);            
            Console.WriteLine($"Result : {result}, enum : {t.ToString()}");
        }
    }
}
```

### 결과
```
Result : True, enum : TEAM1
Result : False, enum : NONE
Result : True, enum : TEAM2
Result : True, enum : 5
```

