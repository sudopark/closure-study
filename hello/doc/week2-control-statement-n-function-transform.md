
## 제어문과 함수형 변환

- 식(expression): 평가될 수 있는 코드
ex) (first) ; first 함수에 인수가 없기때문에 문법에 맞지 않음
- 형식(form): 평가될 수 있는 적법한 식
ex) (first [1 2 3])


### 논리에 따라 흐름 제어하기
- 어떤 값이 불린 true라는 것을 어떻게 알수 있을까? -> true? 함수
(true? true) ; true
(true? false); false
- false를 검사하는 경우는 -> false?
- 값이 없음을 검사하는 경우 -> nil?
-> 그렇다면 부정은, 어떤것의 반대는 어떻게 검사할까? -> not?
: not 함수는 인수가 논리적으로 거짓이면 true를 반환, 그렇지 않으면 false를 반환
(not true) ; false
(not nil) ; true -> nil은 논리적으로 거짓임
(not "hi") ; false -> 값이 있어도 false

#### 비교 연산자
(= :k :k) ;; true
(= :k 2) ;; false
(= `(1 2 3) [1 2 3]) ; true

(not (= x y)) 표현이 길다면 (not= x y)를 이용

### 컬렉션에 사용하는 논리 검사
empty? -> 벡터와, 리스트, 맵, 집합이 비었는지 확인 가능
(empty? [1 2 3]) ;; false
(empty? []) ;; true

;; empty?의 실제 정의
(defn empty? [coll]
    (not (seq coll)))
여기서 seq는 무엇일까?
-> 클로저에는 콜렉션 추상과, 시퀀스 추상이 있음

- collection
백터, 리스트, 맵 처럼 단순 요소를 모아 놓은 것
clojure.lang.IPersistentCollection 인터페이스를 구현(컬렉션을 추상화한) => 존속적이고 불변적인 자료구조
이를 통해 collection은 count, conj, seq와 같은 함수들을 공유

- sequence
seq 함수는 콜렉션을 시퀀스로 바꾸어 준다. -> 시퀀스 추상은 컬렉션을 리스트처럼 순차적으로 다룰 수 있게 해준다.
시퀀스들도 존속적이고 불변이며 first, rest, cons 함수들을 공유한다.
seq 함수는 컬렉션을 받아 시퀀스를 반환하는데 비어있는 경우 nil을 반환한다.

켈렉션을 처리하는 많은 함수들이 이미 seq로 구현하는 경우가 많아 편리한 경우가 있다 -> first의 경우
하지만 비어있지 않음을 검사하는 경우 (not (empty [])) == (not (not (seq [])))에는 바로 seq를 이용하는 것이 관용적이다.
(seq []) ;; nil

every? 
모든 요소에 대한 검사 결과를 반호ks
(every? predicate collection)
(every? (= 1) [1 2 3]) ;; false
(every? odd? [1 1 3])  ;; true

(defn drinkable? [x]
    (= x :drinkme))
(every? drinkable? [:drinkme :drinkme]) ;; true
(every? #(= % :drinkme) [:drinkme :drinkme]) ;; true

not-any?
모든 요소가 거짓인 경우
(not-any? #(= % 1) [1 2 3]) ;; false
(not-any? #(= % 1) [2 3 4]) ;; true

some
진위 함수가 평가한 값이 처음으로 논리적으로 참일때 그 평가한 값을 반환하고 아니면 nil을 반환한다.
(some #(> % 3) [1 2 3 4]) ;; 4

시퀀스에 요소가 있는지 확인할때 집합을 진위 함소로 사용하면 매우 편리하다.
진위 함수가 반환한 값이 nil이나 false가 아니면 논리적으로 참으로 취급되는것을 기억하자, 또한 집합은 요소의 존재 여부를 확인할 수 있는 함수로 사용될 수 있다.
(#{1 2 3} 1) ;; true
(some #{1} [1 2 3 4 5]) ;; 1
(some #{4 5} [1 2 3 4 5]) ;; 4
하지만 논리적으로 거짓인 값에 대해서는 조심해야 한다.
(some #{nil} [nil nil nil]) ;; nil
(some #{false} [false false false]) ;; nil


### 흐름제어 이용하기

if
(if true 1 2) ;; 1
(if false 1 2) ;; 2
(if nil 1 2) ;; 2
(if (= :k :k) 1 2) ;; 1

if-let
if-let은 식을 평가한 결과를 심볼에 바인딩한 후 만약 그 결과가 논리적으로 참이라면 첫번째 인수를 평가하고, 아니면 마지막 인수를 평가한다.
이것은 let을 잠시 쓰고 다음에 if를 쓰는 것보다 더 간결하다.
(let [need-to-grow-small (> 5 3)]
    (if need-to-grow-small 1 2))
;; 1
(if-let [need-to-grow-small (> 5 3)]
    1 2)
;; 1

when
결과가 참일 경우만 처리하고 싶을때, 참일 경우만 본문을 평가한다. 참이 아니면 nil을 반환한다.
(defn drink [need-to-grow-small]
    (when need-to-grow-small "drink bottle"))
(drink true) ;; "drink bottle"
(drink false) ;; nil

when-let
논리검사 결과를 심볼에 바인딩하고, 참이면 본문을 평가하고 아니면 nil을 반환
(when-let [need-to-grow-small] "drink bottle")
; "drink bottle"

cond
조던식을 여러개 쓰고 싶은 경우
cond식은 검사식과 그 검사식이 참일 때 평가될 식의 쌍들을 받는다.
-> 다른 언어의 if/else-if 구문과 비슷함
(let [bottle "drink me"]
    (cond 
        (= bottle "poison") "dont touch"
        (= bottle "drink me") "drink"
        (= bottle "empty") "all gone"))
; "drink"
cond 절에서는 한 검사가 참이면 해당 식이 평가되고 나머지는 무시되기 때문에 순서가 중요하다.

(let [x 5]
    (cond 
        (> x 10) "bigger than 10"
        (> x 4) "bigger than 4"
        (> x 3) "bigger than 3"))
;; "bigger than 4"
(let [x 5]
    (cond 
        (> x 10) "bigger than 10"
        (> x 3) "bigger than 3"
        (> x 4) "bigger than 4"))
;; "bigger than 3"
어떤 검사식도 참이 아니라면 nil이 반환된다.

디폴트절을 넣고 싶으면 맨 마지막에 :else 키워드를 넣는다 -> if/else와 같음
-> :else는 논리적으로 참으로 평가됨 => 고로 참으로 평가되는 아마 문자열이 들어가도 상관없음
(let [x 0]
    (cond 
        (> x 10) "bigger than 10"
        (> x 3) "bigger than 3"
        (> x 4) "bigger than 4"
        :else "some"))
; "some"

cond 검사식ㅇ ㅣ같은 심볼을 반복적으로 검사한다면 좀 더 간결한 식을 사용할 수 있음
검사할 심볼이 같고 그 값을 =로 비교할 수 있는 경우에는 cond 대신에 단축형인 case를 사용할 수 있다.
(let [bottle "drinkme"]
    (case bottle
        "posion" "1"
        "drinkme" "2"
        "empty" "3"))
; "2"

하지만 만약 참인 경우가 없다면 case는 cond와 다르게 동작한다. 참이 없는 경우에 nil을 반환하는 cond와는 달리 case는 참인 절이 없다는 예외를 던진다.
case에 기본값을 제공하려면 맨 마지막에 하나의 식을 주면 되는데, 참인 경우가 하나도 없을 때 이 식이 평가된다,
(let [bottle "dont drinkme"]
    (case bottle
        "posion" "1"
        "drinkme" "2"
        "empty" "3"
        "unknown"))
;; unknown


### 함수를 만드는 함수
함수의 인수를 일부만 적용하고 나중에 나머지를 적용하고자할때 partial 함수를 사용하면 된다.
partial은 클로저의 커링을 하는 방법이다.
커링(currying)은 다중 인수를 갖는 함수를 여러개의 단일 인수 함수들로 연결(chain)하는 방식으로 변환하는 기법을 말한다.
-> 인수를 부분적으로 적용해서 새로운 함수를 만드는 방법

(defn grow [name direction]
    (if (= direction :small) 
        (str name " is growing smaller")
        (str name " is growing bigger")))
-> 여기서 이름은 "alice"로 고정하고 방향만 따로 받는 함수를 partial 함수를 적용해서 만드면
(partial grow "alice")
((partial grow "alice") :small)  ; => "alice is growing smaller"

comp
comp는 여러 험수들을 합성해서 하나의 새로운 함수를 만든다.
(defn toggle-grow [direction]
    (if (= direction :small) :big :small))
(toggle-grow :small) ;; :big

(defn oh-my [direction]
    (str "oh you are growing " direction))

;; 직아졌다 커지는 경우표시
(oh-my (toggle-grow :small)) ; => "oh you are growing bigger"

;; comp 함수를 이용하여 두 함수를 합성
(defn suprise [direction]
    ((comp oh-my toggle-grow) direction))

;; example
(defn add [x y]
    (+ x y))

(def add-5 [x]
    (partial add 5) x)
(def add-5 (partial add 5))

### 구조분해
구조분해는 벡터와 맵 같은 컬렉션에서 특정 요소들에 이름을 붙인다.
(let [[color size] ["blue" "small"]]
    (str "the " color " door is " size))
["blue" "small"] -> color와 size에 값이 할당되었음
구조분해는 바인딩(binding expression)에서 심볼들의 위치에 따라서 어떤 값을 어떤 심볼에 바인딩할지를 결정한다.

(let [x ["blue" "small"]
    color (first x)
    size (last x)]
    (str "the " color " door is " size))
;; 구조분해를 쓰지 않으면 코드가 장황해진다.

구조분해는 중첩된 경우에도 아주 쉽게 처리한다.
(let [[color [size]] ["blue" ["very small"]]]
    (str ....))

구조 분해를 통해 원하는 값을 바로 심볼로 바인딩하는 동시에, 처음 자료구조 전체를 얻고 싶다면 어떻게 해야할까?
이럴때 :as 키워드를 사용하면 된다.
(let [[color [size] :as original] ["blue" ["small"]]]
    {:color color :size size :original original})

구조분해는 맵에도 사용할 수 있다
let 에서 맵의 키에 해당하는 값을 심볼에 바인딩 할 수 있다.
(let [{f1 :f1 f2 :f2}
        {:f1 "red" :f2 "blue"}]
    (str "the flowers are " f1 " and " f2))
    ;; the flowers are red and blue

:or 를 쓰면 맵에 해당하는 요소가 없을때를 위한 디폴트 값을 설정할 수 있다.
(let [{f1 :f1 f2 :f2 :or {f2 "missing"}}
        {:f1 "red"}]
    (str "the flowers are " f1 + " and " f2))
;; the flowers are red and missing

처음 자료구조 전체를 얻으려면 맵에서도 마찬가지로 :as를 쓴다
(let [{f1 :f1 :as all-fs}
        {:f1 "red"}]
    [f1 all-fs])
;; ["red" {:f1 "red"}]

:keys
:keys는 다음에 심볼들의 벡터가 나온다.
(let [{:keys [f1 f2]}
        {:f1 "red" :f2 "blue"}]
    (str ....))

defn으로 함수를 정의할때도 함수의 인수에 대해서 구조분해를 적용할 수 있다.
구조분해는 들어오는 자료구조의 요소를 함수 인수에 바인딩할 뿐만 아니라 그 자료구조가 어떤 형태인지 보여준다는 점에서 유용하다.
(defn flower-colors [colors]
    (str "the flowers are "
    (:f1 colors)
    "and "
    (:f2 colors)))
(flower-colors {:f1 "red" :f2 "blue"})

-> 위 함수를 구조분해를 사용하여 인수가 맵이라는것을 더 명시적으로 선언할 수 있다.
(defn flower-colors [{:keys [f1 f2]}]
    (str ...))


### 지연의 힘
클로저는 보통 콜렉션 이외에 무한한 리스트도 다룰수 있다.
() 이상의 정수 중에서 처음의 다섯개를 뽑아보자
(tak 5 (range) ;; (0 1 2 3 4)
위는 지연 시퀀스라는 것을 이용한다. range 함수가 지연 시퀀스를 반환한다. range 함수에 인수를 하나 주면 끝을 지정할 수 있다.
(range 5) ;; (0 1 2 3 4)
(class (range 5)) ;; clogure.lang.LongRange

끝을 지정하지 않으면 기본으로 범위는 무한이다. 고로 다음을 평가하면 에러가 난다
(range)

take
take는 전체 무한 시퀀스를 평가하는 대신에 필요한 만큼만 평가한다.
(take 10 (range))

repeat
repeat는 반복된 요소로 무한한 시퀀스를 만들어낸다.
(repeat 3 "some") ;; ("some" "some" "some")

range와 마찬가지로 끝을 지정하지 않으면 무한이 된다
(take 2 (repeat 1)) ;; (1 1)

정수로된 난수 시퀀스를 만들기 위해 repeat의 인자로 rand-int를 사용할 수 있다.
(repeat 5 (rand-int 10)) ;; (7 7 7 7 7) -> 결과가 무작위가 아니다.
이경우 repeatedly를 써야한다. repeat는 같은 값을 계속해서 만들어내는 반면 repeatedly는 인수로 받은 함수를 반복적으로 실행

repeatedly는 인수가 없는 함수를 받는다.
(repeatedly 5 #(rand-int 10))

cycle
컬렉션을 인수로 받아 그 콜렉션이 무한으로 반복되는 시퀀스를 반환
(take 3 (cycle [2 3])) ;; (2 3 2)

클로저의 다른 시퀀스 함수들도 지연 시퀀스 대상으로 동작한다.
rest 함수는 지연 시퀀스를 받으면 또 다른 지연 시퀀스를 반환한다.
(take 3 (rest (cycle [1 2]))) ;; (2 1 2)


### 재귀
재귀 함수는 자기 자신을 호출하는 함수이다.
자료구조를 순회하는 아주 우아한 방법

(def adjs ["normal" "too small" "too big" "is swimming"])
(defn alice-is [in out]
    ((if (empty? in)
    out
    (alice-is 
        (rest in)
        (conj out (str "alice is " (first in)))
        ))))
(defn alice-is [input]
    (loop [in input out []]
        (if (empty? in) 
            out
            (recur 
                (rest in)
                (conj (str "alice is " (first in)))
            ))
    )
)

recur를 사용하면 재귀 호출시 스택을 소모하지 않는다. 함수가 수행되기 전에 스택 프레임이 너무 많이 쌓이면 스택 오버플로우 에러가 날 수 있다.
(defn countdown [n]
    (if (= n 0)
        n
        (countdown (- n 1))
    )
)
위의 함수를 recur를 이용하여 재작성하면
(defn countdown [n]
    (if (= n 0)
        n
        (recur (- n 1))
    )
)

recur는 점프할 재귀점을 정의하고 함수 인수에 새로운 값을 넣어 실행하여 매번 호출할때 하나의 스택만을 사용한다.
위 예제에서는 loop가 없어서 재귀점은 함수 자체이다.


### 함수형 프로그래밍에서의 데이터 변환

(def animals [:mouse :duck :dodo :lory :eaglet])
(#(str %) :mouse) ;; ":mouse"

(map #(str %) animals)  ;; (":mouse" ":duck" ":dodo" ":lory" ":eaglet")
반환된 결과는 백터가 아니라 시퀀스이다 => clogure.lang.LazySeq

map 함수는 지연 시퀀스를 반환한다. 지연 시퀀스는 무한 시퀀스를 다를 수 있음을 의미한다.

부수효과
(def animals-print (map #(println %) animals)) ;; 
이경우 출력되는 것이 아무것도 없다. -> 시퀀스가 실제 사용될때 까지는 부수효과가 생기지 않기 때문에
animals-print ;; 순서대로 출력되고 + 마지막줄에 (nil .... )

민약 부수 효과를 강제하고 싶다면 doall을 사용하면 된다.
(def animals-print2 (doall (map #(println %) animals)))
;; 바로 부수효과 출력

animals-print2 ;; (nil ...) 만 출력

map 함수는 인수로 컬렉션을 하나 이상 받을 수 있다. -> 이경우 짧은 콜렉션에 맞춰 종료된다.


### reduce
reduce 함수는 클로저에서 가장 중요하고 기본적인 함수 중의 하나이다.
(reduce + [1 2 3]) ;; 6
(reduce (fn [r x] (+ r (* x x))) [1 2 3])  ;; 14
(reduce (fn [r x] (if (= x nil) x (conj r x))) [1 nil 2 nil 3]) ;; [1 2 3]

reduce는 map과 달리 무한 시퀀스에서 사용할 수 없다.

### 다른 유용한 데이터 처리 함수들
(filter (comp not nil?) [1 nil 2]) ;; (1 2) 문법 틀렸을수도있음
(remove nil? [1 nil 2]) ;; (1 2)

for
for는 처리하고자하는 컬렉션의 요소를 심볼에 바인딩해서 본문에서 처리한다.
(for [animals [1 2 3]]
    (str .. animals))

for구문에서 사용되는 콜렉션이 하나 이상이면 컬렉션들을 중첩하여 순회한다.
(for [animals [:1 :2 :3]
      colors [:r :g]]
    (str (name animals) (name color)))
;; (":1:r" ":1:g"
;;  ":2:r" ":2:g"
;;  ":3:r" ":3:g")

:let 수정자
for 구문 내에서 let 바인딩을 할 수 있다.
(for [animals [:m :d :l]
     colors [:r :g]
     :let [animal-str (str "animal-" (name animals))
            color-str (str "color-" (name colors))
            display-str (str animal-str "-" color-str)]]
        disply-str)

:when 수정자
진위함수가 참일때만 본문이 수행된다.
(
    for [
        animal [:m :d :l]
        color [:r :b]
        :let [
                animal-str (str "animal-" (name animal))
                color-str (str "color-" (name color))
                display-str (str animal-str "-" color-str)
            ]
        :when (= color :blue)
    ]
    display-str
)

flatten
flatten은 중첩된 컬렉션을 받아서 중첩을 제거한 하나의 시퀀스를 반환한다.
(flatten [[:d [:m] [[:l]]]]) ;; (:d :m :l)

리스트를 벡터로 변환
(vec `(1 2 3)) ;; [1 2 3]
(into [] `(1 2 3)) ;; [1 2 3] -> conj로 첫번째 컬렉션에 두번째 컬렉션을 추가한다.

into는 맵을 정렬 맵(sorted map)으로 바꿀때도 사용할 수 있다.
정렬맵은 키-값 쌍들이 키에 따라 정렬되어있다.
(sorted-map :b 2 :a 1 :c 3) ;; {:a 1 :b 2 :c 3}
(into (sorted-map) {...})

또한 쌍들의 백터도 맵으로 바꿀 수 있다.
(into {} [[:a 1] [:b 2] [:c 3]])  ;; {:a 1 :b 2 :c 3}
반대도 가능
(into [] {:a 1 :b 2}) ;; [[:a 1] [:b 2]]

partition
컬렉션의 요소들을 묶음으로 나누는데 유용
(partition 3 [1 2 3 4 5 6 7]) ;; ((1 2 3) (4 5 6))
요소가 충분하지 않으면 묶을수 있는 만큼 묶는다. 남은 요소들도 묶고 싶다면
(partition-all 3 [1 2 3 4]) ;; ((1 2 3) 94)

partition-by 
하나의 함수를 받아서 컬렉션의 모든 요소에 그 함수를 적용하고 값이 변할때마다 새로운 묶음을 만들어낸다
(partition-by #((= % 6)) [1 2 3 4 5 5 6 7 8 9])  ;; ((1 2 3 4 5) (6) (7 8 9))
