---
title: "프로그래머스 - 기능개발"
comments: true
author: jaehyun
date: 2020-02-26
tags:
  - 알고리즘

---

### 프로그래머스 - 기능개발

---

#### 문제 설명
프로그래머스 팀에서는 기능 개선 작업을 수행 중입니다. 각 기능은 진도가 100%일 때 서비스에 반영할 수 있습니다.  

또, 각 기능의 개발속도는 모두 다르기 때문에 뒤에 있는 기능이 앞에 있는 기능보다 먼저 개발될 수 있고, 이때 뒤에 있는 기능은 앞에 있는 기능이 배포될 때 함께 배포됩니다.  

먼저 배포되어야 하는 순서대로 작업의 진도가 적힌 정수 배열 progresses와 각 작업의 개발 속도가 적힌 정수 배열 speeds가 주어질 때 각 배포마다 몇 개의 기능이 배포되는지를 return 하도록 solution 함수를 완성하세요.  

#### 제한 사항
작업의 개수(progresses, speeds배열의 길이)는 100개 이하입니다.  
작업 진도는 100 미만의 자연수입니다.  
작업 속도는 100 이하의 자연수입니다.  
배포는 하루에 한 번만 할 수 있으며, 하루의 끝에 이루어진다고 가정합니다. 예를 들어 진도율이 95%인 작업의 개발 속도가 하루에 4%라면 배포는 2일 뒤에 이루어집니다.  

#### 입출력 예
|progresses|speeds|return|
|:---:|:---:|:---:|
|[93,30,55]|[1,30,5]|[2,1]|

#### 입출력 예 설명
첫 번째 기능은 93% 완료되어 있고 하루에 1%씩 작업이 가능하므로 7일간 작업 후 배포가 가능합니다.  
두 번째 기능은 30%가 완료되어 있고 하루에 30%씩 작업이 가능하므로 3일간 작업 후 배포가 가능합니다. 하지만 이전 첫 번째 기능이 아직 완성된 상태가 아니기 때문에 첫 번째 기능이 배포되는 7일째 배포됩니다.  
세 번째 기능은 55%가 완료되어 있고 하루에 5%씩 작업이 가능하므로 9일간 작업 후 배포가 가능합니다.  

따라서 7일째에 2개의 기능, 9일째에 1개의 기능이 배포됩니다.  

#### 코드 

``` java
import java.util.*;

public class P_functionDevelop {

    public static int[] solution(int[] progresses, int[] speeds) {
        int [] answer = {};
        int [] temp = new int[progresses.length];
        List<Integer> list = new ArrayList<Integer>();

        // 매칭되는 task마다 진행도를 매칭하여 몇일이 걸려야하는지 계산
        for(int i=0; i<progresses.length; i++) {
            int count = 0;
            int sum = progresses[i];
            while (sum < 100) {
                sum += speeds[i];
                count++;
            }
            temp[i] = count;
        }

        // 기준값을 설정하여 뒤에 자신보다 빨리 끝난 일이 있으면 자기의 카운트 ++
        // 자신보다 큰 값이 나오면 list에 add하고 피봇값 변경
        int pivot = temp[0];
        int count = 1;
        for(int j=1; j<temp.length; j++) {
            if(temp[j] <= pivot) {
                count++;
            } else {
                list.add(count);
                pivot = temp[j];
                count = 1;
            }
        }
        list.add(count);

        answer = new int[list.size()];
        for(int k=0; k<list.size(); k++) {
            answer[k] = list.get(k);
        }

        return answer;
    }

    public static void main (String[] args) {
        int[] progresses = {40, 93, 30, 55, 60, 65};
        int[] speeds = {60, 1, 30, 5, 10, 7};
        // [1, 2, 3]

        // [2,1]
        solution(progresses, speeds);

    }

}

```

