## Item3: 일방향 @ManyToOne은 얼마나 효율적일까?

Author -|-O< Book 관계는 동일, 다만 Author에서는 Book(s)을 관리하지 않고 Book 에서 Author에 대해서만 관리하는 경우이다.

---

다양하고 복잡하게 설명되어 있지만, 크게 중요하지 않아 생략

---

### 결론: (자식 측에서의) 일방향 @ManyToOne은 효율이 좋다. 양방향 @OneToMany가 필요하지 않은 상황이라면 이 방법을 고려하라.