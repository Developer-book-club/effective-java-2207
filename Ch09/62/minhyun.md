사실 DB를 사용하다보면 숫자부터 모~든 데이터를 문자열로 관리하고 있는 것을 많이 보는데 잘못된 것이다.

사용 용도에 따라 정확한 타입을 선택하자 (문자열로 퉁치지 말자)

---

# 문자열은 다른 값 타입을 대신하기에 적합하지 않다.

- 열거 타입을 대신하기 적합하지 않다.
- 혼합 타입을 대신하기 적합하지 않다.
    - (ex) 두 요소를 구분해주는 문자 ‘#’ 등 (minhyun#weight)
- 권한을 표현하기 적합하지 않다.

# 정리

- 더 적합한 데이터 타입이 있을 경우 문자열을 쓰고 싶은 유혹을 뿌리쳐라.