embed 프롬프트...
#### **Instructions**:

1. Analyze the **provided problem definition** and **logical steps** written by the user. If logical steps are incomplete or unclear, infer additional steps based on the problem description.
2. Divide the **latest user's code** into sections corresponding to these logical steps. Separate each step with **a single blank line** for clarity.
3. At the start of each step, include a comment indicating the step number (e.g., `// < Step 1 >`) and provide a concise explanation of the step's intended purpose.
4. Add **step-specific guiding comments** below each step header:
    - Describe what the code is doing in this step.
    - Highlight potential problems, ambiguities, or misalignments between the code and the logical steps.
    - Include **thought-provoking questions** to help the user understand the issue.
    - If appropriate, suggest better approaches or corrections as comments without modifying the code.
5. Ensure the response adheres to the **output format** below.

---

**Output Format**:

- Present the **user's code with added comments** as the output.
- Divide the code into steps, separated by **one blank line**.
- Use `// < Step x >` to label each step and provide guidance as comments under the label.
- Do **not modify the original code**—all suggestions and corrections should appear as comments.

---

**Example Input**:
- Problem definition:  
    Mirko works in a sugar factory as a delivery boy. He has just received an order: he has to deliver exactly N kilograms of sugar to a candy store on the Adriatic coast. Mirko can use two types of packages: 3-kilogram and 5-kilogram packages. Mirko would like to take as few packages as possible. For example, if he has to deliver 18 kilograms of sugar, he could use six 3-kilogram packages. But, it would be better to use three 5-kilogram packages and one 3-kilogram package, totaling four packages. Help Mirko by finding the minimum number of packages required to transport exactly N kilograms of sugar.
    
- Logical steps:
    
    1. Take the input `n` representing the required kilograms.
    2. Calculate the remainder when dividing `n` by 5 and 3.
    3. If `n` is divisible by 5, compute the number of packages required.
    4. If the remainder is divisible by 3, compute the remaining packages.
    5. If neither condition applies, output `-1`.

- User code:
```cpp
#include <iostream>  using namespace std;  int main(){ int n = 0; cin >> n; int modFive = n % 5 ; int modThree = n % 3 ; if(modFive % 3 == 0) return modFive + modFive % 3; if(modThree == 0) return modThree; cout << "-1"; return 0; } }
```




```cpp
#include <iostream>  using namespace std;   
int main() {      // < Step 1 >     // Take the input `n` representing the required kilograms.     // [Guide]: Ensure that `n` meets the problem's constraints (e.g., `n` should be a non-negative integer).     int n = 0;      cin >> n;       // < Step 2 >     // Calculate the remainders when dividing `n` by 5 and 3.     // [Guide]: The variables `modFive` and `modThree` store the remainders.     // Check if these values are being used correctly in the subsequent steps.     int modFive = n % 5;      int modThree = n % 3;       // < Step 3 >     // Check if `n` is divisible by 5 or can be adjusted using 3-kilogram packages.     // [Guide]: The condition `if(modFive % 3 == 0)` might not align with the problem's requirements.     // Consider if this condition correctly checks whether the remainder can be handled using 3-kilogram packages.     if (modFive % 3 == 0)          return modFive + modFive % 3;       // < Step 4 >     // Handle the case where `n` is divisible by 3.     // [Guide]: `if(modThree == 0)` correctly handles cases divisible by 3.     // However, it does not account for cases where both conditions fail.     if (modThree == 0)          return modThree;       // < Step 5 >     // Handle cases where `n` cannot be exactly divided using the packages.     // [Guide]: Outputting `-1` is correct, but ensure this covers all edge cases (e.g., when `n` is negative).     cout << "-1";      return 0;  }
```













gpt가 완전히 수정한 코드(답변)
```cpp
#include <iostream>
using namespace std;

// < Step 1 > 몇 킬로그램을 배달할지 입력받는다.
// [Suggested improvement]: 입력값에 대한 유효성 검사를 추가해 음수나 잘못된 값이 들어오지 않도록 한다.
int main() {
    int n;
    cin >> n;

    int count = 0; // 봉지 개수를 저장할 변수


// < Step 2 > 5로 나눴을 때와 3으로 나눴을 때 남은 무게를 처리한다.
// [Suggested improvement]: 무게가 정확히 나누어지지 않을 경우를 처리하기 위해 반복문을 사용하여 조건을 점검한다.
    while (true) {
        // 5로 나눌 수 있는 경우
        if (n % 5 == 0) {
            count += n / 5; // 5로 나눈 몫을 봉지 개수에 추가
            break;
        }
        // 5로 나눌 수 없는 경우, 3kg 봉지를 하나 사용
        n -= 3;
        count++;

        // n이 음수가 되면 정확히 나눌 수 없는 경우
        if (n < 0) {
            count = -1;
            break;
        }
    }


// < Step 3 > 계산된 봉지 개수를 출력한다.
// [Suggested improvement]: 사용자가 올바르지 않은 무게를 입력한 경우 명확한 에러 메시지를 출력하도록 한다.
    cout << count << endl;

    return 0;
}

```

- If user's input language is NOT an English(cf. Korean), CHANGE GPT's OUTPUT LANGUAGE into USER'S one.
- Check GPR's response one more time if the NEW code is same as previous code escept steps and comments that GPT added.
