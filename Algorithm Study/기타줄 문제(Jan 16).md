input:
	n 끊어진 줄 개수 m 기타줄 브랜드
	각 브랜드 패키지가격 낱개가격

output: 
	적어도 n개의 기타줄을 사기 위해 필요한 돈의 최솟값

풀이: 
필요한 기타줄 n개
각 브랜드 패키지 가격 <- 패키지, 6개! 

n '* cheepest 낱개 
cheepest 브랜드 패키지


packageCount = 6으로 나눴을 때 몫
eachCount = 6 mod

4 = 6 없음 4 네 개

10 = 6 하나 4 네 개
가장 싼 낱개 cheepestEach
가장 싼 패키지 cheepestPackage

 a) cheepestPackage* packageCount + cheepestEach* eachCount

-> 근데 그냥 패키지를 더 사는게 나을수도 있잖아! (낱개 mod 가 좀 많거나 애매하게 가격 있을때, or 뭉탱이가 싼 경우... )
b) cheepestPackage* ( packageCount + 1)
-> 낱개로만 살 때 쌀 수도 있는 가능성도 고려.

a와 b 비교해서 cheepest 출력

예시1: 
15 1
100 40

15/6 = 2...3

2 * 100 + 40 * 3 = 320
(2+1) * 100 = 300

예시2: 
7 2
10 3
12 2

7/6 = 6...1

6 * 10 + 2 = 12
(1+1) * 10 = 20

결과코드
```cpp
#include <iostream>
using namespace std; // for cin

int main(){
	int cheapestPackage = 1001, cheapestEach = 1001; // 가장 싼 패키지/낱개 가격
	int packageCount = 0, eachCount = 0; // n을 6으로 나눴을 때의 몫과 나머지
	int n = 0, m = 0; // 끊어진 줄 개수 , 기타줄 브랜드
	
	cin >> n >> m;
	packageCount = n / 6;
	eachCount = n % 6;
	
	int brandPackagePrice; // 각 브랜드별 패키지 가격
	int brandEachPrice; // 각 브랜드별 낱개 가격
	
	int accurateCalculate = 0, roundOffCalculate = 0, onlyEachCalculate = 0;
	
	for (int i = 0; i < m; i++){
		cin >> brandPackagePrice;
		cin >> brandEachPrice;
		
		if (brandPackagePrice <= cheapestPackage) // 가장 싼 패키지 가격 찾기
		cheapestPackage = brandPackagePrice;
		
		if (brandEachPrice <= cheapestEach) // 가장 싼 개별 가격 찾기
		cheapestEach = brandEachPrice;
	}
	
	accurateCalculate = cheapestPackage * packageCount + cheapestEach * eachCount;
	roundOffCalculate = cheapestPackage * (packageCount + 1);
	onlyEachCalculate = n * cheapestEach;
	
	int result = 0;
	if (accurateCalculate <= roundOffCalculate)
		result = accurateCalculate;
	else
		result = roundOffCalculate;
	if (result > onlyEachCalculate)
		result = onlyEachCalculate;
	
	cout << result << endl;
	return 0;

}
```