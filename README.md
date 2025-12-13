# CS-algorithm-study

### 팀원 소개

| 이름  | github 아이디 | 담당 주차 | 비고 |
| ----- | ------------- | --------- | ---- |
| 조성화(알고) |    [hwacho9](https://github.com/hwacho9)           | 5,10         | 일본대학 유학생(26졸), 전공생   |
| 안현우(알고) |               | -         | 4,11    |
| 이재엽(알고) |               | -         | 1,8    |
| 장형원(알고) |               | -         | 2,9    |
| 허용준 |               | -         | 3    |
| 문성철 |               | -         | 6    |
| kevin |               | -         | 7    |

### **1. 역할 소개**

-   **👨‍🏫 발표자 (1명)**
    -   **주요 역할:** 해당 주차의 주제를 가장 깊이 있게 학습하여 스터디원들에게 핵심 내용을 발표하고 설명합니다.
    -   **이렇게 준비해주세요!**
        -   스터디원들이 이해하기 쉽도록 발표 자료(PPT, Notion, 마크다운 등)를 준비합니다.
        -   레포지토리에 발표 자료를 올립니다.
        -   단순 정보 나열이 아닌, "왜 이것이 중요한가?"에 초점을 맞춰 설명합니다.
-   **🤔 토론/질문자 (6명)**
    -   **주요 역할:** 발표 내용을 들으며 비판적인 시각으로 질문을 던지고, 자유로운 토론을 이끌어냅니다. 스터디의 깊이를 더하는 역할입니다.
    -   **이렇게 준비해주세요!**
        -   발표 주제를 미리 학습하고, **질문을 최소 2개 이상** 준비합니다.

### **2. 주간 스터디 진행 순서**

1. **알고리즘 문제 풀이(알고리즘 풀이 참가자만) :** 알고리즘 문제를 풀고, 코드를 공유합니다.
2. **발표 :** 2명의 발표자가 준비한 내용을 설명합니다.
3. **심층 토론 및 Q&A :** 5명의 토론자가 준비한 질문을 중심으로 모든 스터디원이 자유롭게 의견을 나눕니다.
4. **정리 및 마무리 :** 다음 주 주제와 역할 담당자를 공지하며 마무리합니다.

---

### 📚 전체 커리큘럼 (수정 할 수도 있습니다.)

| 기간   | 대주제       | 주차별 세부 주제                                                                                                                                                                                                 |
| ------ | ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1–3주  | 자료구조     | • 1주: 시간복잡도(Big-O), 배열, 연결 리스트, 스택, 큐(재엽)<br>• 2주: 해시 테이블(충돌 해결 방법 포함), 트리, 힙(형원)<br>• 3주: 그래프(DFS/BFS 기본 탐색), 각 자료구조별 시간복잡도 총정리표 및 실습(성화) |
| 4–5주  | 네트워크     | • 4주: OSI 7계층 vs TCP/IP 4계층, TCP와 UDP, 3-way-handshake(현우)<br>• 5주: HTTP/HTTPS, REST API, CDN·Caching·Reverse Proxy (실무 서비스 설계)(용준)                                           |
| 6–7주  | 운영체제     | • 6주: 프로세스 vs 스레드, CPU 스케줄링 알고리즘(성철)<br>• 7주: 메모리 관리(페이징·세그먼테이션), 스레드 동기화(세마포어·뮤텍스), 교착상태(Deadlock) 발생 조건 및 해결 방법(kevin)                      |
| 8–9주  | 데이터베이스 | • 8주: 관계형 데이터베이스, 정규화, 트랜잭션(ACID), 격리 수준(재엽)<br>• 9주: 인덱스 원리(B-Tree 등), SQL 최적화, 실행 계획 분석(형원)                                                               |
| 10–11주 | 디자인 패턴  | • 10주: 디자인 패턴의 개념과 필요성, 싱글톤·팩토리·스트래티지 패턴(성화)<br>• 11주: 옵저버·프록시 패턴, MVC·MVVM 등 아키텍처 패턴(현우)                                                               |
| 12주   | 종합 정리     | • 전 주차 핵심 복습 및 CS 기술 면접 대비<br>• 실습 과제 또는 미니 프로젝트 발표 (예: 간단한 서비스 설계에 패턴/DB/네트워크 요소 반영)                                                   |


---

### 참고 자료
https://github.com/VSFe/Tech-Interview

| 분야    | 교재/참고 사이트                                                |
| ----- | -------------------------------------------------------- |
| 자료구조  | 『이것이 코딩테스트다』,  |
| 네트워크  |                 |
| 운영체제  | 『운영체제 공룡책(Operating System Concepts)』                    |
| DB    |                                |
| 디자인패턴 |        |

--- 

### 알고리즘
### **🔹 第1週: 基本概念 & 配列探索**

- 📌 **理論学習**: Arrays & Hashing (ハッシュ、配列探索の手法)
    
    ![スクリーンショット 2025-03-05 17.06.26.png](attachment:ed326b6c-80bb-44ee-bd0c-925a51d99206:スクリーンショット_2025-03-05_17.06.26.png)
    

📌 **問題演習**:

1. https://leetcode.com/problems/contains-duplicate/description/
2. https://leetcode.com/problems/valid-anagram/description/
3. https://leetcode.com/problems/two-sum/description/
4. https://leetcode.com/problems/group-anagrams/description/
5. https://leetcode.com/problems/top-k-frequent-elements/description/
6. https://leetcode.com/problems/encode-and-decode-tinyurl/description/
7. https://leetcode.com/problems/product-of-array-except-self/description/
8. https://leetcode.com/problems/valid-sudoku/description/
9. https://leetcode.com/problems/longest-consecutive-sequence/description/
- 追加問題: Leetcode75

### **🔹 第2週: ツーポインター & スタック**

📌 **理論学習**: Two Pointers & Stack (Slow/Fast, LIFO)

📌 **問題演習**:

- LeetCode 75:
    - https://leetcode.com/problems/valid-palindrome/description/
    - https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/
    - https://leetcode.com/problems/3sum/
    - https://leetcode.com/problems/container-with-most-water/
    - https://leetcode.com/problems/trapping-rain-water/description/
    - https://leetcode.com/problems/valid-parentheses/description/
    - https://leetcode.com/problems/min-stack/description/
    - https://leetcode.com/problems/evaluate-reverse-polish-notation/description/
    - https://leetcode.com/problems/daily-temperatures/description/
    - https://leetcode.com/problems/car-fleet/
    - https://leetcode.com/problems/largest-rectangle-in-histogram/description/
- 追加問題:

---

### **🔹 第3週: 二分探索 & スライディングウィンドウ**

📌 **理論学習**: Binary Search (Lower/Upper Bound)、Sliding Window

📌 **問題演習**:

- LeetCode 75:
    - https://leetcode.com/problems/binary-search/
    - https://leetcode.com/problems/search-a-2d-matrix/description/
    - https://leetcode.com/problems/koko-eating-bananas/description/
    - https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/
    - https://leetcode.com/problems/search-in-rotated-sorted-array/
    - https://leetcode.com/problems/time-based-key-value-store/
    - https://leetcode.com/problems/median-of-two-sorted-arrays/
    - https://leetcode.com/problems/best-time-to-buy-and-sell-stock/
    - https://leetcode.com/problems/longest-substring-without-repeating-characters/description/
    - https://leetcode.com/problems/longest-repeating-character-replacement/description/
    - https://leetcode.com/problems/permutation-in-string/
    - https://leetcode.com/problems/minimum-window-substring/
    - https://leetcode.com/problems/sliding-window-maximum/
- 追加問題:


### 💡 기타 규칙 및 약속

-   **소통 채널:** 슬랙 (공지 및 빠른 소통), Github (스터디 자료 아카이빙)
-   **스터디 시간:** 매주 일요일 09:00
-   **기본 규칙:** 정해진 역할은 책임감을 가지고 수행합니다. 불참 시 최소 하루 전까지는 미리 공유하기!
-   **발표 자료 올리기:** 발표 자료를 Github에 올립니다.
-   **질문 준비:** 발표 주제를 미리 학습하고, 질문을 최소 2개 이상 준비합니다.
-   **토론 준비:** 발표 내용을 들으며 비판적인 시각으로 질문을 던지고, 자유로운 토론을 이끌어냅니다.
-   **정리 및 마무리:** 다음 주 주제와 역할 담당자를 공지하며 마무리합니다.
