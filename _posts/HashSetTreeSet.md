# HashSet 과 TreeSet - 순서 X, 중복 X

HashSet 

- Set 인터페이스를 구현한 대표적인 컬렉션 클래스

- 순서를 유지하려면 LinkedHashSet 클래스를 사용하면 된다.

TreeSet

- 범위 검색과 정렬에 유리한 컬렉션 클래스

- HashSet 보다 데이터 추가, 삭제에 시간이 더 걸림

- HashSet - boolean add(Object o)

- HashSet 은 객체를 저장하기 전에 기존에 같은 객체가 있는 지 확인한다.

- 같은 객체가 없으면 저장(true 반환) 하고 있으면 저장하지 않는다(false 반환).

- boolean add(Object o) 는 저장할 객체의 equals() 와 hashCode() 를 호출

eqauls() 와 hashCode() 가 오버라이딩 되어 있어야함

```
import java.util.HashSet;
import java.util.Objects;

public class HashSetTest {
    public static void main(String[] args) {
        HashSet set = new HashSet();
        set.add(new Person("David", 10));
        set.add(new Person("David", 10));

        System.out.println(set); //[David:10]

    }
}

class Person {
    String name;
    int age;

    Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public boolean equals(Object obj) {
        if (obj instanceof Person) {
            Person tmp = (Person) obj;
            return name.equals(tmp.name) && age == tmp.age;

        }
        return false;
    }

    public int hashCode() {
//        return (name + age).hashCode();
        return Objects.hash(name, age);
    }

    public String toString() {
        return name + ":" + age;
    }
}
```

- HashSet - hashCode() 의 오버라이딩 조건
1. 동일 객체에 대해 hashCode() 를 여러 번 호출 해도 같은 값을 반환해야 한다.
2. equals() 로 비교해서 true 를 얻은 두 객체의 hashCode() 값은 일치해야 한다.

→ equals() 로 비교한 결과가 false 인 두 객체의 hashCode() 값이 같을 수도 있다.

그러나 성능 향상을 위해 가능하면 서로 다른 값을 반환하도록 작성하자.

- TreeSet - 범위 검색과 정렬에 유리

- 범위 검색과 정렬에 유리한 이진 검색 트리(binary search tree) 로 구현

LinkedList 처럼 각 요소(node) 가 나무(tree) 형태로 연결된 구조

- 이진 트리는 모든 노드가 최대 두 개의 하위 노드를 갖음(부모 -자식관계)

- 이진 검색 트리는 부모보다 작은 값을 왼쪽에, 큰 값은 오른쪽에 저장한다.

- TreeSet - 데이터 저장과정 boolean add(Object o)

→ 루트부터 트리를 따라 내려가며 값을 비교, 작으면 왼쪽 크면 오른쪽에 저장

**TreeSet - 범위 검색 subSet(), headSet(), tailSet()**

| 메서드                                                 | 설명                                                         |
| ------------------------------------------------------ | ------------------------------------------------------------ |
| SortedSet subSet(Object fromElement, Object toElement) | 범위 검색 (fromElement 와 toElement 사이) 결과를 반환한다. (끝 범위인 toElement 는 범위에 포함되지 않음) |
| SortedSet headSet(Object toElement)                    | 지정된 객체보다 작은 값의 객체들을 반환한다.                 |
| SortedSet tailSet(Object fromElement                   | 지정된 객체보다 큰 값의 객체들을 반환한다.                   |

- 트리 순회(전위, 중위, 후위)

- 이진 트리의 모든 노드를 한 번씩 읽는 것을 트리 순회라고 한다.

- 전위, 중위, 후위 순회법이 있으며 중위 순회하면 오름차순으로 정렬된다.

전위 순회법 → 부모 노드, 왼쪽 노드, 오른쪽 노드 순으로 읽는다.

후위 순회법 → 왼쪽 노드, 오른쪽 노드, 부모 노드 순으로 읽는다.

중위 순회법 → 왼쪽 노드, 부모 노드, 오른쪽 노드 순으로 읽는다.

→ 중위 순회법을 사용하면 정렬된 결과를 얻을 수 있다.