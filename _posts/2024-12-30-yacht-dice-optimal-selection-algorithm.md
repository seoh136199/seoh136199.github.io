---
title: "[Pancht] 요트 다이스 최적 선택 알고리즘을 구현한 방법"

categories:
  - Development Log
tags:
  - [Pancht, Algorithm, Dynamic Programming, Bitmasking]

toc: true
toc_sticky: true

date: 2024-12-31
last_modified_at: 2024-12-31
---

# • 서론 •

## 왜 하게 되었는가

요트 다이스의 룰을 차용한 Pancht라는 게임에 AI 대전 모드를 넣게 되었다. 이 AI는 숙련된 플레이어의 실력에 준하는, 또는 그 이상의 실력을 갖춰야 고티어 플레이어와 유의미한 대결이 가능할 것이다.<br>
숙련된 플레이어의 의사 결정을 모방해서 로직을 직접 작성하는 방법도 있겠지만 코드가 매우 복잡해질 것으로 예상되었고, 이후 설명할 방식에 비하면 획득할 수 있는 최종 점수의 기댓값이 낮을 것 같았다. 따라서 플레이 종료 시점까지의 미래를 전부 계산하여 항상 최적의 선택을 하는 AI 알고리즘을 짜서 구현하기로 했다. (저티어 플레이어를 위한 실력 조절에 대해서는 추후 포스팅 예정)

요트 다이스 게임을 하며 플레이어가 주사위를 굴린 직후, 마주할 수 있는 경우의 수를 계산해보자.<br>

- 카테고리가 총 13개라면, 각 카테고리가 채워졌는지 여부에 따른 경우의 수는 2^13 = **8192가지**이다. (정확히 계산하려면 모든 카테고리가 채워진 상태에는 더 이상 주사위를 굴리지 않으므로 1을 빼야 하지만 편의를 위해 생략했다)<br>
- 주사위를 최대 두 번 리롤할 수 있으므로, 한 턴에 주사위를 굴리는 횟수는 최대 3번이다. 따라서 주사위를 굴리는 횟수로 인한 경우의 수는 **3가지**이다.<br>
- 그리고 6면체 주사위를 5개 굴리므로, 중복 조합으로 계산했을 때 주사위에서 나올 수 있는 경우의 수는 6H5 = **252가지**이다.

즉, 8192 x 3 x 252 = **6,193,152가지**의 경우의 수에 대한 모든 최적 행동을 구하는 알고리즘을 짜면, 게임의 어디에서든 구해놓은 최적 행동을 이용해서 플레이가 가능할 것이다.
<br><br>

## 어떻게 구현했는가

1. **모든 가능한 행동을 하나씩 다 수행**해본다.
   - 즉, 카테고리 선택을 하거나, 주사위 일부 또는 전부를 리롤한다.
2. 그 행동을 취한 후 주사위를 굴렸을 때 나올 수 있는 **모든 주사위의 조합에 대해 시뮬레이션하여, 얻을 수 있는 점수의 기댓값을 계산**한다.
   - 카테고리 선택을 한 경우: 다음 턴이 시작되어 주사위를 새로 굴리므로 주사위 눈의 조합 252개 전체에 대해 시뮬레이션한다.
   - 리롤하는 경우: 리롤하기로 택한 주사위에 따라 나올 수 있는 모든 경우의 수에 대해 시뮬레이션 한다.
3. 위에서 구한 점수 기댓값 중 **가장 높은 기댓값을 갖는 행동**을 선택한다.
   - 즉, 점수 기댓값이 가장 높은 상태로 이동하는 행동을 취한다.

주사위를 굴릴 때마다 위 작업을 반복하면 항상 최선의 선택을 할 수 있을 것이다.<br>
그러나 각 행동에서 얻을 수 있는 최고 점수를 알기 위해서는, 해당 시점부터 게임 종료까지 발생할 수 있는 모든 행동과 주사위 경우의 수를 전부 탐색해야 한다. 이 방식을 나이브하게 구현하면, 코드 실행에 상상도 못 할 정도로 오랜 시간이 걸릴 것이다. 따라서 최적화가 필요하다.
<br><br>

## 어떻게 최적화했는가

위에서 구한 약 6백만 가지의 경우의 수에는 **방향성**이 있다. 게임을 플레이할 때는 항상 카테고리가 점점 많이 채워지는 방향으로 진행하기 때문이다.<br>
즉, 어떤 상황에서 최적 행동을 구하기 위해 '현재 상황보다 카테고리가 적게 채워진 상태'의 기댓값은 알 필요가 없다. 그런 상태는 이미 과거에 지나쳤거나, 현재 상황과는 무관한 상태이기 때문이다.

하지만 '현재 상황보다 카테고리가 많이 채워진 상태', 즉 **미래에 발생할 수 있는 어떤 상태의 기댓값은 현재의 최적 행동을 구하기 위해 꼭 필요한 정보**다.
게임의 종료에 가까운 상태의 기댓값은, 그보다 종료에 덜 가까운 상태의 최적 행동 계산에 필요하다. 또한 어떤 상태의 기댓값은 여러 번 참조될 수 있다.<br>
따라서 어떤 상태의 기댓값을 계산하게 되면 그 값을 저장해두고, 나중에 필요할 때는 다시 계산하지 않고 저장해둔 값을 재활용함으로써 수행 시간을 획기적으로 줄일 수 있다. 바로 **다이나믹 프로그래밍**(DP)을 사용하는 것이다.
<br><br>

## 주의 사항

- 여기서 보관하는 값은 행동에 대한 기댓값이 아닌, **상태에 대한 기댓값**이다. 어떤 상태에서 행동을 취함으로써 또 다른 상태로 이동하게 된다. 이동할 수 있는 상태들의 기댓값들을 비교하여 어느 상태로 이동할지 결정하는 것이 곧 어떤 행동을 선택할지 결정하는 것과 같다.
- **어떤 상태의 기댓값**이란 그 상태에서 게임을 시작했다고 가정하고, 게임 종료에 도달했을 때까지 얻을 수 있는 점수의 기댓값이라는 의미이다. 그 상태가 되기 이전에 얻은 점수는 포함하지 않는다.
  <br><br>

---

# • PanchtAI.h •

```cpp
#ifndef PANCHT_AI_H
#define PANCHT_AI_H

enum Category {
    ACES, TWOS, THREES, FOURS, FIVES, SIXES,
    CHOICE, THREE_OF_A_KIND, FOUR_OF_A_KIND, FULL_HOUSE,
    SMALL_STRAIGHT, LARGE_STRAIGHT, YACHT,
    NONE
};

struct DicesInfo {
    int dices[5];
    int& operator[](int idx) { return dices[idx]; }
};

constexpr int ANS_6_POW[5] = { 1, 6, 36, 216, 1296 };
constexpr int ANS_6_POW_5 = 7776;
constexpr int ANS_2_POW_5 = 32;
constexpr int ANS_2_POW_13 = 8192;

constexpr int CATEGORY_CNT = 13;
constexpr int DICE_CNT = 5;
constexpr int DICE_CASE_CNT = 252; // = 6H5 = 10C5

extern const char* categoryNames[];
extern int8_t bestAction[ANS_2_POW_13][3][DICE_CASE_CNT];

extern double aiScoreWeight;

extern void initComb();
extern void initArrays();
extern int getEarnedScore(Category category, DicesInfo dicesInfo, bool forAI);
extern int convertDicesInfoToInt(DicesInfo& diceInfo);
extern void convertDiceIntToInfo(int idx, DicesInfo& dices);
extern int convertActionInfoToInt(Category category, int rerollMask);
extern void convertActionIntToInfo(int actionInt, Category& category, int& rerollMask);
extern void calBestActionAndScore(int categoryMask, int rerolledCnt, DicesInfo dicesInfo);

#endif
```

최적 선택 로직이 필요한 코드에서 공용으로 사용할 헤더 파일을 만들었다. 이 파일에는 카테고리 열거형과 주사위 눈 정보 구조체, 그리고 일부 상수, 배열, 함수 선언이 포함되어 있다.
각 함수의 구현 세부 내용은 뒤에서 소개할 `PanchtAI.cpp` 코드 설명에서 다루겠다.
<br><br>

---

# • PanchtAI.cpp의 배열들 •

## combTable[]

```cpp
static int combTable[20][20];
```

조합 연산이 필요한 부분에서 빠르게 참조할 수 있도록, 미리 계산한 값을 저장해둔 배열이다. `combTable[n][m]`을 참조하면 nCm의 값을 얻을 수 있다.
<br><br>

## bestAction[]

```cpp
int8_t bestAction[ANS_2_POW_13][3][DICE_CASE_CNT];
```

`bestAction` 배열은 위에서 말한 약 6백만 가지의 **상태 각각에 대한 최적 행동**을 저장하는 3차원 배열이다.<br>
최적 행동을 하나의 정수로 인코딩하는 방법은 이후 나올 함수 구현 설명에서 자세히 다루겠다.

- 첫 번째 인자: 각 카테고리 선택 여부를 정수로 변환한 값 (0~8191)
- 두 번째 인자: 해당 턴에 리롤한 횟수를 나타낸 값 (0~2)
- 세 번째 인자: 주사위 5개의 상태를 하나의 정수로 변환해 나타낸 값 (0~251)
  <br><br>

## bestScore[]

```cpp
double bestScore[ANS_2_POW_13][3][DICE_CASE_CNT];
```

`bestScore` 배열은 약 6백만 가지의 **상태 각각에서 얻을 수 있는 최대 기댓값**을 저장한다. 각 인자들의 의미는 `bestAction` 배열과 동일하다.<br>
게임 후반으로 갈수록 각 상태의 기댓값 차이가 미미해지므로 소수점 아래 값이 중요해진다. 처음에는 이 값을 `int`로 저장했었는데, 이렇게 하면 기댓값의 정수 부분이 동일한 행동들 중 코드상에서 먼저 계산하는 행동을 선택하게 된다. 그 결과, 리롤을 하지 않고 카테고리 선택을 성급하게 해버리는 경향이 있었다. 이 문제는 자료형을 `double`로 변경하며 기댓값 손실을 방지하여 해결되었다.
<br><br>

## avgScore[]

```cpp
double avgScore[ANS_2_POW_13];
```

`avgScore` 배열은 2^13 = 8192가지의 카테고리 선택 여부에 따라 **턴 시작 시점에서 얻을 수 있는 기댓값**을 저장한다.
<br><br>

## avgScore2[]

```cpp
double avgScore2[ANS_2_POW_13][3][DICE_CASE_CNT][ANS_2_POW_5];
```

`avgScore2` 배열은 약 6백만 가지의 **상태 각각에서 리롤하는 모든 선택지에 대한 기댓값**을 저장한다.

- 첫 번째 인자: 각 카테고리 선택 여부를 정수로 변환한 값 (0~8191)
- 두 번째 인자: 해당 턴에 리롤한 횟수를 나타낸 값 (0~2)
- 세 번째 인자: 주사위 5개의 상태를 하나의 정수로 변환해 나타낸 값 (0~251)
- 네 번째 인자: 리롤할 주사위 선택 여부를 하나의 정수로 나타낸 값 (0~31)
  <br><br>

---

# • PanchtAI.cpp의 함수들 •

## initComb() & comb()

```cpp
void initComb() {
    for (int i = 0; i < 20; i++) {
        combTable[i][0] = 1;
        for (int j = 1; j <= i; j++) {
            combTable[i][j] = combTable[i - 1][j - 1] + combTable[i - 1][j];
        }
    }
}

int comb(int n, int k) {
    if (k > n || k < 0) return 0;
    return combTable[n][k];
}
```

처음 프로그램 실행 시 `initComb()`를 한 번 호출하여 미리 `combTable` 배열을 초기화한다.
이후 어디서든 `comb()`를 호출해 조합 연산의 결과를 빠르게 사용할 수 있도록 작성했다.
<br><br>

## initArrays()

```cpp
void initArrays() {
    for (int i = 0; i < ANS_2_POW_13; i++) {
        avgScore[i] = -1.0;

        for (int j = 0; j < 3; j++) {
            for (int k = 0; k < DICE_CASE_CNT; k++) {
                bestAction[i][j][k] = -1;
                bestScore[i][j][k] = -1.0;

                for (int l = 0; l < ANS_2_POW_5; l++) {
                    avgScore2[i][j][k][l] = -1.0;
                }
            }
        }
    }
}
```

위에서 설명한 여러 DP 배열의 모든 원소를 `-1`로 초기화하는 함수이다. `initComb()`와 마찬가지로 프로그램 처음 실행 시 한 번 호출해야 한다.<br>
이후, 이 배열들을 참조할 때 값이 `-1`이라면 아직 계산되지 않았음을 의미하므로 새로 계산해야 하고, `-1`이 아니라면 이미 계산된 것이므로 그대로 재활용한다.
<br><br>

## getEarnedScore()

```cpp
int getEarnedScore(Category category, DicesInfo dicesInfo, bool forAI = true) {
    if (ACES <= category && category <= SIXES) {
        int score = 0;
        for (int i = 0; i < 5; i++) {
            if (dicesInfo.dices[i] == category + 1) score += category + 1;
        }
        if (forAI) return score * aiScoreWeight;
        return score;
    }
    int totSum = 0;
    for (int i = 0; i < 5; i++) totSum += dicesInfo.dices[i];
    int numFreq[7] = { 0 };
    for (int i = 0; i < 5; i++) numFreq[dicesInfo.dices[i]]++;
    if (category == CHOICE) {
        return totSum;
    }
    if (category == THREE_OF_A_KIND) {
        for (int i = 1; i <= 6; i++) {
            if (numFreq[i] >= 3) return totSum;
        }
        return 0;
    }
    if (category == FOUR_OF_A_KIND) {
        for (int i = 1; i <= 6; i++) {
            if (numFreq[i] >= 4) return totSum;
        }
        return 0;
    }
    if (category == FULL_HOUSE) {
        bool hasThree = false, hasTwo = false, hasFive = false;
        for (int i = 1; i <= 6; i++) {
            if (numFreq[i] == 3) hasThree = true;
            if (numFreq[i] == 2) hasTwo = true;
            if (numFreq[i] == 5) hasFive = true;
        }
        if (hasThree && hasTwo || hasFive) return totSum;
        return 0;
    }
    if (category == SMALL_STRAIGHT) {
        for (int i = 1; i <= 3; i++) {
            bool isSmallStraight = true;
            for (int j = i; j < i + 4; j++) {
                if (numFreq[j] == 0) {
                    isSmallStraight = false;
                    break;
                }
            }
            if (isSmallStraight) return 15;
        }
        return 0;
    }
    if (category == LARGE_STRAIGHT) {
        for (int i = 1; i <= 2; i++) {
            bool isLargeStraight = true;
            for (int j = i; j < i + 5; j++) {
                if (numFreq[j] == 0) {
                    isLargeStraight = false;
                    break;
                }
            }
            if (isLargeStraight) return 30;
        }
        return 0;
    }
    if (category == YACHT) {
        for (int i = 1; i <= 6; i++) {
            if (numFreq[i] == 5) return 50;
        }
        return 0;
    }
    return 0;
}
```

선택한 카테고리와 주사위 눈 정보를 넘기면 **그 카테고리에서 해당 주사위 눈으로 얻을 수 있는 점수**를 반환하는 함수이다.
<br><br>

## convertDicesInfoToInt()

```cpp
int convertDicesInfoToInt(DicesInfo& dicesInfo) {
    int freq[6] = { 0 };
    for (int i = 0; i < DICE_CNT; i++) {
        freq[dicesInfo[i] - 1]++;
    }

    int idx = 0, remain = 5;
    for (int i = 0; i < 6; i++) {
        int x = freq[i];
        for (int skip_val = 0; skip_val < x; skip_val++) {
            int ways = comb(remain - skip_val + 5 - i - 1, 5 - i - 1);
            idx += ways;
        }
        remain -= x;
    }
    return idx;
}
```

**주사위 다섯 개의 눈 정보를 받아 0부터 251 사이의 정수로 매핑**하는 함수이다. 이를 통해 주사위 눈 정보를 배열의 인덱스로 직접 사용할 수 있게 해준다.<br>
원래 주사위 5개에서 나올 수 있는 경우의 수는 6^5 = 7776가지이지만, 이 게임에서는 주사위 눈의 순서가 중요하지 않다. 순서를 무시하고 오름차순으로 정렬했을 때는 경우의 수가 252가지로 줄어든다. 이 점을 활용해 0~251 범위로 오름차순 매핑하였다.
<br><br>

## convertDiceIntToInfo()

```cpp
void convertDiceIntToInfo(int idx, DicesInfo& dices) {
    int freq[6] = { 0 };
    int remain = 5;

    for (int i = 0; i < 6; i++) {
        int x = 0;
        while (true) {
            if (remain - x < 0) break;

            int ways = comb(remain - x + (5 - i) - 1, (5 - i) - 1);

            if (ways == 0) {
                x++;
                continue;
            }

            if (ways > idx) {
                freq[i] = x;
                remain -= x;
                break;
            }
            else {
                idx -= ways;
                x++;
            }
        }
    }

    dices = { 6, 6, 6, 6, 6 };
    int pos = 0;
    for (int face = 0; face < 6; face++) {
        for (int count = 0; count < freq[face]; count++) {
            dices[pos++] = face + 1;
        }
    }
}
```

**매핑된 주사위 눈 정수를 받아 다시 5개의 주사위 눈 정보로 변환**하는 함수이다. 항상 오름차순으로 정렬된 주사위 눈 정보를 반환한다.
<br><br>

## convertActionInfoToInt()

```cpp
int convertActionInfoToInt(Category category, int rerollMask) {
    if (category != NONE) return category;
    return 12 + rerollMask;
}
```

카테고리 정보와 리롤 주사위 정보를 각각 정수로 입력받아, **행동을 하나의 정수로 압축**하여 반환한다. 하나의 행동에는 '카테고리 선택' 또는 '리롤' 중 한 가지만 포함될 수 있으므로, 이 함수를 호출할 때는 무조건 `category`를 `NONE`으로 하거나 `rerollMask`를 `0`으로 설정해야 한다.

반환값은 0~44 범위를 갖는다.

- 0~12: 특정 카테고리 선택
- 13~44: 주사위 일부 또는 전체를 리롤
  <br><br>

## convertActionIntToInfo()

```cpp
void convertActionIntToInfo(int actionInt, Category& category, int& rerollMask) {
    if (actionInt <= 12) {
        category = (Category)actionInt;
        rerollMask = 0;
        return;
    }
    category = NONE;
    rerollMask = actionInt - 12;
}
```

**압축된 행동 정수를 다시 카테고리 정보 또는 리롤 주사위 정보로 변환**하는 함수이다.
<br><br>

## getAvgScore(categoryMask)

```cpp
double getAvgScore(int categoryMask) {
    if (avgScore[categoryMask] >= 0) return avgScore[categoryMask];

    double sum = 0;
    for (int diceInfoInt = 0; diceInfoInt < DICE_CASE_CNT; diceInfoInt++) {
        DicesInfo dicesInfo;
        convertDiceIntToInfo(diceInfoInt, dicesInfo);
        calBestActionAndScore(categoryMask, 0, dicesInfo);
        sum += bestScore[categoryMask][0][diceInfoInt];
    }
    return avgScore[categoryMask] = sum / DICE_CASE_CNT;
}
```

위에서 언급한 avgScore 배열에 대응하는 DP 함수이다. 주어진 카테고리 선택 여부에서 **턴을 시작한 시점에 얻을 수 있는 기댓값**을 반환하는 함수이다. 이를 위해 가능한 252가지 주사위 결과를 모두 시뮬레이션하고, 각 결과의 최대 기댓값들의 평균을 반환함으로써 원하는 값을 얻는다.
<br><br>

## getAvgScore(categoryMask, rerolledCnt, dicesInfo, rerollMask)

```cpp
double getAvgScore(int categoryMask, int rerolledCnt, DicesInfo dicesInfo, int rerollMask) {
    int dicesInfoInt = convertDicesInfoToInt(dicesInfo);
    double& crAvgScore = avgScore2[categoryMask][rerolledCnt][dicesInfoInt][rerollMask];
    if (crAvgScore >= 0) return crAvgScore;

    int rerollIdx[5];
    int rerollCnt = 0;
    for (int i = 0; i < DICE_CNT; i++) {
        if ((rerollMask & (1 << i)) == 0) continue;
        rerollIdx[rerollCnt++] = i;
    }

    int rerollCaseCnt = 1;
    for (int i = 0; i < rerollCnt; i++) rerollCaseCnt *= 6;

    double sum = 0;
    for (int caseInt = 0; caseInt < rerollCaseCnt; caseInt++) {
        DicesInfo newDices = dicesInfo;

        int tmp = caseInt;
        for (int j = 0; j < rerollCnt; j++) {
            int roll = (tmp % 6) + 1;
            tmp /= 6;
            newDices[rerollIdx[j]] = roll;
        }

        calBestActionAndScore(categoryMask, rerolledCnt + 1, newDices);
        int newDicesInt = convertDicesInfoToInt(newDices);
        sum += bestScore[categoryMask][rerolledCnt + 1][newDicesInt];
    }

    return crAvgScore = sum / rerollCaseCnt;
}
```

위에서 언급한 `avgScore2` 배열에 대응하는 DP 함수이다. 각 카테고리 선택 여부, 리롤 횟수, 주사위 눈 정보, 그리고 리롤할 주사위 정보를 받아서 **해당 리롤을 수행했을 때 얻게 되는 기댓값**을 반환하는 함수이다. 마찬가지로 리롤 후 가능한 주사위의 결과를 모두 시뮬레이션하고, 각 결과의 최대 기댓값들의 평균을 반환함으로써 원하는 값을 얻는다.
<br><br>

## calBestActionAndScore()

```cpp
void calBestActionAndScore(int categoryMask, int rerolledCnt, DicesInfo dicesInfo) {
    int dicesInfoInt = convertDicesInfoToInt(dicesInfo);
    int8_t& crBestAction = bestAction[categoryMask][rerolledCnt][dicesInfoInt];
    double& crBestScore = bestScore[categoryMask][rerolledCnt][dicesInfoInt];

    sort(dicesInfo.dices, dicesInfo.dices + DICE_CNT);

    //이미 계산한/불러온 케이스인 경우 계산 결과 반환
    if (crBestAction != -1) return;

    int remainCategoryCnt = 0;
    for (int i = 0; i < CATEGORY_CNT; i++) {
        if ((categoryMask & (1 << i)) == 0) remainCategoryCnt++;
    }

    //카테고리가 1개 남았고 3번째 굴림인 경우 해당 카테고리 반환
    if (remainCategoryCnt == 1 && rerolledCnt == 2) {
        Category onlyCategory = NONE;
        for (int i = 0; i < CATEGORY_CNT; i++) {
            if ((categoryMask & (1 << i)) != 0) continue;
            onlyCategory = (Category)i;
            break;
        }
        crBestAction = convertActionInfoToInt(onlyCategory, 0);
        crBestScore = getEarnedScore(onlyCategory, dicesInfo, true);
        return;
    }

    //========== 모든 경우의 수에 대해 최대 기댓값을 얻는 액션 찾기 ==========

    Category bestCategory = NONE;
    int bestRerollMask = 0;
    double bestAvgScore = 0;

    //카테고리를 선택하는 경우
    for (int i = 0; i < CATEGORY_CNT; i++) {
        if ((categoryMask & (1 << i)) != 0) continue;
        Category crCategory = (Category)i;

        double crAvgScore = getEarnedScore(crCategory, dicesInfo, true) + getAvgScore(categoryMask | (1 << i));

        if (crAvgScore > bestAvgScore) {
            bestAvgScore = crAvgScore;
            bestCategory = crCategory;
            bestRerollMask = 0;
        }
    }

    if (rerolledCnt == 2) {
        crBestAction = convertActionInfoToInt(bestCategory, 0);
        crBestScore = bestAvgScore;
        return;
    }

    //리롤하는 경우
    for (int rerollMask = 1; rerollMask < ANS_2_POW_5; rerollMask++) {
        double crAvgScore = getAvgScore(categoryMask, rerolledCnt, dicesInfo, rerollMask);

        if (crAvgScore > bestAvgScore) {
            bestAvgScore = crAvgScore;
            bestCategory = NONE;
            bestRerollMask = rerollMask;
        }
    }

    crBestAction = convertActionInfoToInt(bestCategory, bestRerollMask);
    crBestScore = bestAvgScore;
}
```

위에서 언급한 `bestAction` 배열과 `bestScore` 배열에 대응하는 DP 함수이자 가장 중요한 함수이다. 각 카테고리 선택 여부, 리롤 횟수, 주사위 눈 정보를 인자로 받는다. 즉, 주사위를 굴린(또는 리롤한) 직후 상태를 총 3개의 인자로써 나타낸 것이다.

플레이어가 취할 수 있는 행동은 크게 두 가지가 있다.

1. 빈 카테고리를 선택한다
2. (리롤 횟수가 2보다 작다면) 주사위 일부 또는 전체를 골라 리롤한다

함수는 이 두 종류의 가능한 모든 행동을 시도해 본 뒤, 각각의 행동을 통해 이동하게 될 다음 상태의 기댓값들을 모두 계산 및 비교한다. 그리고 그 중 그 **기댓값이 가장 큰 행동과 그 점수**를 동시에 반환한다.
<br><br>

---

# • PanchtPlayer.cpp •

## 소스코드

```cpp
#include <iostream>
#include <algorithm>
#include <fstream>
#include <stdint.h>
#include <iomanip>
#include <ctime>
#include "PanchtAI.h"
using namespace std;

DicesInfo getRandomDices() {
    DicesInfo dicesInfo;
    for (int i = 0; i < DICE_CNT; i++) {
        dicesInfo[i] = rand() % 6 + 1;
    }
    sort(dicesInfo.dices, dicesInfo.dices + DICE_CNT);
    return dicesInfo;
}

DicesInfo getRandomRerolledDices(DicesInfo dicesInfo, int& rerollMask) {
    for (int i = 0; i < DICE_CNT; i++) {
        if (rerollMask & (1 << i)) {
            dicesInfo[i] = rand() % 6 + 1;
        }
    }
    sort(dicesInfo.dices, dicesInfo.dices + DICE_CNT);
    return dicesInfo;
}

string diceInfoString(DicesInfo dicesInfo) {
    string str = "";
    for (int i = 0; i < DICE_CNT; i++) {
        str += (dicesInfo[i] + '0');
        if (i != DICE_CNT - 1) str += ", ";
    }
    return str;
}

void printRerollMask(int rerollMask) {
    string str = "rerollMask: ";
    for (int i = 0; i < DICE_CNT; i++) {
        str += (rerollMask & (1 << i)) ? '1' : '0';
    }
    cout << str << '\n';
}


bool selectAction(int rerolledCnt, int& score, int& numScore, DicesInfo& dicesInfo, int& categoryMask,
    Category& bestCategory, int& bestRerollMask) {

    calBestActionAndScore(categoryMask, rerolledCnt, dicesInfo);
    int dicesInfoInt = convertDicesInfoToInt(dicesInfo);
    int bestActionInt = bestAction[categoryMask][rerolledCnt][dicesInfoInt];

    convertActionIntToInfo(bestActionInt, bestCategory, bestRerollMask);

    if (bestCategory == NONE) {
        printRerollMask(bestRerollMask);
        return false;
    }

    int crScore = getEarnedScore(bestCategory, dicesInfo, false);
    cout << "선택한 카테고리: " << categoryNames[bestCategory] << '\n';
    cout << "획득한 점수: " << crScore << "\n\n";

    score += crScore;
    if (ACES <= bestCategory && bestCategory <= SIXES) numScore += crScore;

    categoryMask |= (1 << bestCategory);
    return true;
}

int playGame() {
    int categoryMask = 0, score = 0, numScore = 0;

    for (int turn = 1; turn <= CATEGORY_CNT; turn++) {
        cout << "<" << turn << "번째 턴 시작>" << '\n';

        Category bestCategory;
        int bestRerollMask;

        DicesInfo dicesInfo = getRandomDices();
        cout << "첫 번째 롤: " << diceInfoString(dicesInfo) << '\n';
        if (selectAction(0, score, numScore, dicesInfo, categoryMask, bestCategory, bestRerollMask)) continue;

        dicesInfo = getRandomRerolledDices(dicesInfo, bestRerollMask);
        cout << "두 번째 롤: " << diceInfoString(dicesInfo) << '\n';
        if (selectAction(1, score, numScore, dicesInfo, categoryMask, bestCategory, bestRerollMask)) continue;

        dicesInfo = getRandomRerolledDices(dicesInfo, bestRerollMask);
        cout << "세 번째 롤: " << diceInfoString(dicesInfo) << '\n';
        selectAction(2, score, numScore, dicesInfo, categoryMask, bestCategory, bestRerollMask);
    }

    if (numScore >= 63) score += 35;
    cout << "최종 점수: " << score << "\n\n";
    return score;
}

int main() {
    srand(time(NULL));
    initComb();
    initArrays();

    int crScore = playGame();
    cout << "최종 점수: " << crScore << "\n";

    return 0;
}
```

이 코드를 실행하면 요트 다이스 게임 플레이 시뮬레이션을 수행한다. 실제 플레이 방식을 모방하면서, 행동을 선택해야 할 때마다 `calBestActionAndScore()` 함수를 호출해서 최적 행동을 결정한다.<br>
카테고리 선택 여부와 리롤할 주사위는 비트마스킹을 활용해 각각 하나의 정수로 표현했다.
<br><br>

## 실행 결과

```
<1번째 턴 시작>
첫 번째 롤: 2, 2, 5, 5, 6
rerollMask: 11001
두 번째 롤: 1, 1, 3, 5, 5
rerollMask: 11100
세 번째 롤: 2, 5, 5, 6, 6
선택한 카테고리: Choice
획득한 점수: 24

<2번째 턴 시작>
첫 번째 롤: 1, 1, 1, 2, 4
rerollMask: 00011
두 번째 롤: 1, 1, 1, 1, 3
rerollMask: 00001
세 번째 롤: 1, 1, 1, 1, 3
선택한 카테고리: Aces
획득한 점수: 4

<3번째 턴 시작>
첫 번째 롤: 1, 1, 2, 2, 6
rerollMask: 11001
두 번째 롤: 2, 2, 2, 3, 4
rerollMask: 00011
세 번째 롤: 1, 2, 2, 2, 2
선택한 카테고리: Twos
획득한 점수: 8

<4번째 턴 시작>
첫 번째 롤: 1, 3, 4, 5, 5
rerollMask: 10010
두 번째 롤: 3, 3, 4, 4, 5
rerollMask: 10100
세 번째 롤: 3, 3, 3, 4, 5
선택한 카테고리: Threes
획득한 점수: 9

<5번째 턴 시작>
첫 번째 롤: 1, 2, 4, 5, 5
rerollMask: 11100
두 번째 롤: 1, 3, 5, 5, 5
rerollMask: 11000
세 번째 롤: 4, 5, 5, 5, 5
선택한 카테고리: Fives
획득한 점수: 20

<6번째 턴 시작>
첫 번째 롤: 1, 2, 3, 3, 5
rerollMask: 10100
두 번째 롤: 1, 2, 3, 3, 5
rerollMask: 10100
세 번째 롤: 1, 2, 3, 4, 5
선택한 카테고리: LargeStraight
획득한 점수: 30

<7번째 턴 시작>
첫 번째 롤: 1, 3, 3, 4, 5
rerollMask: 11000
두 번째 롤: 1, 3, 3, 4, 5
rerollMask: 11000
세 번째 롤: 2, 3, 4, 4, 5
선택한 카테고리: SmallStraight
획득한 점수: 15

<8번째 턴 시작>
첫 번째 롤: 2, 3, 5, 5, 5
rerollMask: 11000
두 번째 롤: 2, 5, 5, 5, 5
rerollMask: 10000
세 번째 롤: 5, 5, 5, 5, 5
선택한 카테고리: Yacht
획득한 점수: 50

<9번째 턴 시작>
첫 번째 롤: 2, 2, 3, 3, 5
rerollMask: 11111
두 번째 롤: 1, 2, 4, 5, 6
rerollMask: 11011
세 번째 롤: 2, 2, 3, 4, 6
선택한 카테고리: FullHouse
획득한 점수: 0

<10번째 턴 시작>
첫 번째 롤: 2, 3, 4, 6, 6
rerollMask: 11100
두 번째 롤: 3, 5, 6, 6, 6
rerollMask: 11000
세 번째 롤: 3, 5, 6, 6, 6
선택한 카테고리: Sixes
획득한 점수: 18

<11번째 턴 시작>
첫 번째 롤: 1, 2, 2, 2, 5
rerollMask: 10001
두 번째 롤: 2, 2, 2, 2, 3
rerollMask: 00001
세 번째 롤: 2, 2, 2, 2, 4
선택한 카테고리: FourOfAKind
획득한 점수: 12

<12번째 턴 시작>
첫 번째 롤: 1, 3, 4, 4, 6
rerollMask: 11001
두 번째 롤: 3, 4, 4, 4, 6
rerollMask: 10001
세 번째 롤: 3, 4, 4, 4, 4
선택한 카테고리: Fours
획득한 점수: 16

최종 점수: 241
```

플레이어가 실제로 전략적인 선택을 하는 듯한 모습을 확인했다. 지금까지 짠 코드가 예상대로 동작했음을 알 수 있다.<br>
테스트로 5000회 실행했을 때의 평균 점수는 **205.695점**으로, 준수한 결과가 나왔다.<br>
(참고: DP로 최적화를 했음에도 불구하고 이 코드를 한 번 실행하는 데에 5시간 40분이 소요되었다.)
<br><br>

---

# • 가중치 세팅 •

그러나, 최적 선택 로직에 **보너스 점수 획득**이 반영되지 않았다는 한 가지 문제가 남아 있다. 물론 실제 게임 플레이 시뮬레이션에서 최종 점수를 계산할 때는 보너스 점수를 추가해 주지만, 행동 선택 로직에서는 이 보너스를 무시하고 있기 때문이다.<br>
즉, 카테고리 중 Aces부터 Sixes까지의 획득 점수의 가치가 실제보다 과소평가되고 있다고 볼 수 있다.

실제로 보너스를 고려해서 완벽한 최적 선택을 구현할 수도 있겠지만, 그러면 코드가 매우 복잡해지고 실행 시간도 훨씬 길어질 것이다. 따라서 최적 행동을 선택할 때, **Aces부터 Sixes의 획득 점수에 1보다 큰 가중치를 부여**하여 최적 선택의 근사치를 얻는 방식을 시도해 보았다.<br>
최적의 가중치를 찾기 위해 가중치를 변화시키면서 5천회씩 플레이하여 평균 점수를 구했다.

```
가중치: 1.0, 평균 점수: 205.695
가중치: 1.5, 평균 점수: 208.052
가중치: 2.0, 평균 점수: 211.364
가중치: 2.5, 평균 점수: 211.856
가중치: 3.0, 평균 점수: 211.476
가중치: 3.5, 평균 점수: 211.819
가중치: 4.0, 평균 점수: 209.800
가중치: 4.5, 평균 점수: 209.331
가중치: 5.0, 평균 점수: 207.742
가중치: 5.5, 평균 점수: 206.187
가중치: 6.0, 평균 점수: 203.849
```

예상대로 가중치가 1부터 일정값에 도달할 때까지는 평균 점수가 높아졌다가, 그 후로는 다시 낮아지는 경향성이 보인다. 가중치 2.5와 3.5에서 평균 점수가 가장 높고, 3.0에서는 살짝 낮아졌는데, 이는 5천 회의 시뮬레이션 실행으로는 이상치의 영향을 받았기 때문이라고 판단된다.
결국 **가중치를 3.0으로 결정**함으로써 로직 구현이 완료되었다.

(참고: 이 게임은 한 판에서 발생하는 랜덤 요소가 매우 많기 때문에 5천 회 정도의 테스트 횟수는 부족하며, 천만 회 정도는 테스트해야 통계적으로 안정적인 결과를 얻을 수 있다는 사실을 뒤늦게 알았다.)
<br><br>

---

# • 저장 및 불러오기 •

## 저장하기

매번 플레이할 때마다 5시간 40분씩 코드를 돌릴 수는 없다. 따라서 `bestAction` 배열의 결괏값들을 미리 계산해서 저장해 둔 뒤, 이를 불러와서 사용하기로 했다.

```cpp
int main() {
    initComb();
    initArrays();

    aiScoreWeight = 3.0f;

    ofstream outFile("bestAction.bin", ios::binary);
    if (!outFile.is_open()) {
        printf("bestAction.bin 파일 오픈 실패\n");
        return 1;
    }

    printf("bestAction.bin 파일 오픈 성공\n");

    static int8_t data[ANS_2_POW_13 * 3 * DICE_CASE_CNT];

    for (int i = ANS_2_POW_13 - 1; i >= 0; i--) {
        for (int j = 0; j < 3; j++) {
            for (int k = 0; k < DICE_CASE_CNT; k++) {
                DicesInfo dicesInfo;
                convertDiceIntToInfo(k, dicesInfo);
                calBestActionAndScore(i, j, dicesInfo);

                int64_t bestActionInt = bestAction[i][j][k];

                int idx = i * (3 * DICE_CASE_CNT) + j * DICE_CASE_CNT + k;
                data[idx] = bestActionInt;
            }
        }
        //printf("진행률: %.2f%%\n", (i + 1) / (float)ANS_2_POW_13 * 100);
        printCategoryMask(i);

        time_t t = time(nullptr);
        tm* now = localtime(&t);
        cout << put_time(now, "  %Y/%m/%d %H:%M:%S") << '\n';
    }

    outFile.write(reinterpret_cast<const char*>(data), sizeof(data));
    outFile.close();

    return 0;
}
```

3차원 배열인 `bestAction`을 펼쳐 1차원 배열로 만든 뒤 바이너리 파일에 저장했다. 이 과정을 수행하는 데에도 마찬가지로 5시간 40분이 소요되었다.
<br><br>

## 불러오기

```cpp
void loadBestActionData() {
    ifstream inFile("bestAction.bin", ios::binary);
    if (!inFile.is_open()) {
        printf("bestAction.bin 파일 오픈 실패\n");
        exit(1);
    }
    printf("bestAction.bin 파일 오픈 성공\n");

    static int8_t data[ANS_2_POW_13 * 3 * DICE_CASE_CNT];

    inFile.read(reinterpret_cast<char*>(data), sizeof(data));
    inFile.close();

    for (int i = 0; i < ANS_2_POW_13; i++) {
        for (int j = 0; j < 3; j++) {
            for (int k = 0; k < DICE_CASE_CNT; k++) {
                int idx = i * (3 * DICE_CASE_CNT) + j * DICE_CASE_CNT + k;
                bestAction[i][j][k] = data[idx];
            }
        }
    }

    printf("데이터 불러오기 완료\n\n");
    return;
}
```

저장된 바이너리 파일을 다시 읽어 1차원 배열에 불러온 후, 이를 3차원 배열에 재배치했다.
이제 프로그램이 처음 실행될 때 `loadBestActionData()`를 한 번 호출하면, `bestAction` 배열의 모든 원소가 -1이 아닌 유효 상태가 된다. 그 결과, `calBestActionAndScore()`를 호출했을 때 원하는 값을 즉시 얻을 수 있다.
<br><br>

---

# • 후기 •

보너스를 완벽히 고려한 로직이 아닌 점은 아쉽지만, 200점대로 꽤 준수한 평균 점수를 내는 로직 구현에 성공한 것에 만족한다. 이 AI를 이용해서 뽑아낸 각종 통계를 그래프로 시각화한 자료를 추후 포스팅하겠다.
