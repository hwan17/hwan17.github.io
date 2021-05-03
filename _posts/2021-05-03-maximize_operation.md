--- 
title: 수식 최대화
excerpt: 카카오 2020년 인턴십 문제 풀이

categories:
  - Algorithm
tags:
  - [Algorithm]

toc: true
toc_sticky: true

date: 2021-05-03
last_modified_at: 2021-05-03
---
 
# 수식 최대화

### 문제 링크
https://programmers.co.kr/learn/courses/30/lessons/67257

-, +, * 연산자의 순서를 임의로 설정하여 수식의 결과로 나올 수 있는 값중 절댓값이 가장 큰 값 출력


### 구상
1. re 라이브러리를 사용하여 숫자와 연산자로 분리시킨다. # ['100', '-', '200', '*', '300', '-', '500', '+', '20']
2. 연산자 순서를 미리 리스트로 정의해 놓는다.
   
   cases = [('-','+','*'), ('-','*','+'), 
         ('+','-','*'), ('+','*','-'),
         ('*','-','+'), ('*','+','-')]

3. 선택된 연산자 순서에 맞게 1에서 불러온 리스트에서 연산자가 걸리면, 전에 입력되어 있던 숫자와 다음 입력될 숫자를 연산하여 append한다.
4. 리스트의 길이가 1 == 연산을 모두 수행하고 결과가 나왔을 때 기존의 숫자와 절댓값을 비교하여 큰 값으로 교체한다.
   
### 코드

    def solution(expression):

        import re

        cases = [('-','+','*'), ('-','*','+'), 
            ('+','-','*'), ('+','*','-'),
            ('*','-','+'), ('*','+','-')]
        
        splited_ex = re.findall('[(\d]+|[-+*]', expression)
        
        cand = 0
        for case in cases:
            test_ex2 = [] + splited_ex
            test_ex = []
            while len(test_ex2) != 1:
                for op in case:
                    flag = False
                    for num_op in test_ex2:
                        if flag:
                            temp_op = test_ex.pop()
                            temp_num = test_ex.pop()
                            test_ex.append(str(eval(temp_num+temp_op+num_op)))
                            flag = False
                        elif num_op == op:
                            flag = True
                            test_ex.append(num_op)
                        else:
                            test_ex.append(num_op)
                    test_ex2 = test_ex
                    test_ex = []
            cand = max(cand, abs(int(test_ex2[0])))

        return cand