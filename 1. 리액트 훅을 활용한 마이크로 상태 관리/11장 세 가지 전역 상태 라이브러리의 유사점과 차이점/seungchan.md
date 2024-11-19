# 복잡한 상태가 주어졌을때 어떻게 처리할까?

## 1. 정규화

- 중첩된 데이터 구조를 평탄화하여 관리

```javascript
// 중첩된 구조 (비추천)
const [posts, setPosts] = useState({
  1: {
    id: 1,
    title: "첫 게시글",
    comments: [
      { id: 1, text: "댓글1" },
      { id: 2, text: "댓글2" },
    ],
  },
});

// 정규화된 구조 (추천)
const [posts, setPosts] = useState({
  posts: {
    1: { id: 1, title: "첫 게시글", commentIds: [1, 2] },
  },
  comments: {
    1: { id: 1, text: "댓글1" },
    2: { id: 2, text: "댓글2" },
  },
});
```

## 정규화

- 데이터의 중복을 줄이고 데이터의 정확성을 보장하는 것
- 리액트에서는 데이터의 중복을 줄이고, 상태 업데이트를 용이하게 하는 것

```javascript
1.중복제거
// 중복이 있는 구조
const state = {
  posts: [
    {
      id: 1,
      title: "게시글 1",
      author: {
        id: 1,
        name: "김철수",
        email: "kim@example.com", // 중복 데이터
      },
    },
    {
      id: 2,
      title: "게시글 2",
      author: {
        id: 1,
        name: "김철수",
        email: "kim@example.com", // 중복 데이터
      },
    },
  ],
};

// 정규화된 구조
const state = {
  posts: [
    {
      id: 1,
      title: "게시글 1",
      authorId: 1,
    },
    {
      id: 2,
      title: "게시글 2",
      authorId: 1,
    },
  ],
  authors: {
    1: {
      id: 1,
      name: "김철수",
      email: "kim@example.com", // 한 곳에서만 관리
    },
  },
};



// 정규화 전
const state = {
  orders: [
    {
      orderId: 1,
      productId: 100,
      productName: "노트북", // productId에 종속
      productPrice: 1000000, // productId에 종속
      quantity: 2,
      totalPrice: 2000000
    }
  ]
}

// 적용 후
const state = {
  orders: [
    {
      orderId: 1,
      productId: 100,
      quantity: 2,
      totalPrice: 2000000
    }
  ],
  products: {
    100: {
      id: 100,
      name: "노트북",
      price: 1000000
    }
  }
}


2.상태 업데이트 용이
// 정규화되지 않은 구조에서 저자 정보 업데이트
function updateAuthorEmail(authorId, newEmail) {
  setState(prevState => ({
    ...prevState,
    posts: prevState.posts.map(post => {
      if (post.author.id === authorId) {
        return {
          ...post,
          author: {
            ...post.author,
            email: newEmail
          }
        };
      }
      return post;
    })
  }));
}

// 정규화된 구조에서 저자 정보 업데이트
function updateAuthorEmail(authorId, newEmail) {
  setState(prevState => ({
    ...prevState,
    authors: {
      ...prevState.authors,
      [authorId]: {
        ...prevState.authors[authorId],
        email: newEmail
      }
    }
  }));
}

```

## 정규화의 핵심 원칙:

1.단일 진실 공급원 (Single Source of Truth)

- 각 데이터는 한 곳에서만 관리
- 다른 곳에서는 참조로 연결

2. 의존성 최소화

- 데이터 간의 불필요한 의존성 제거
- 독립적인 업데이트 가능

3. 참조 무결성

- ID를 통한 관계 설정
- 연관된 데이터의 일관성 유지

## 2. 하나의 큰 상태를 작은 상태로 나누어 관리

```javascript
// 하나의 큰 상태 (비추천)
const [formState, setFormState] = useState({
  username: '',
  email: '',
  isValid: false,
  isSubmitting: false,
  errors: {},
  touched: {}
});

// 분리된 상태 (추천)
const [username, setUsername] = useState('');
const [email, setEmail] = useState('');
const [isValid, setIsValid] = useState(false);
const [isSubmitting, setIsSubmitting] = useState(false);
const [errors, setErrors] = useState({});
const [touched, setTouched] = useState({});

단점
- 상태관의 연관성이 떨어져 보임
장점
- 커스텀 훅 같은 로직의 재사용이 쉬워짐
- 컴포넌트 리렌더링 최적화가 쉬워짐
- 각 상태의 업데이트가 독립적
```

```javascript
리렌더링 최적화
// 큰 상태일 때 - name이 바뀌면 email 관련 UI도 리렌더링됨
const handleNameChange = (newName) => {
  setForm(prev => ({
    ...prev,
    name: newName
  }));
};

// 분리된 상태 - name만 변경되면 name 관련 UI만 리렌더링
const handleNameChange = (newName) => {
  setName(newName);
};



function ParentComponent() {
  const [name, setName] = useState("");
  const [email, setEmail] = useState("");

  console.log("Parent rendered");

  return (
    <div>
      <NameComponent name={name} onNameChange={setName} />
      <EmailComponent email={email} onEmailChange={setEmail} />
    </div>
  );
}

// React.memo로 감싸서 최적화
const NameComponent = React.memo(({ name, onNameChange }) => {
  console.log("Name component rendered");
  return (
    <input
      value={name}
      onChange={(e) => onNameChange(e.target.value)}
    />
  );
});

const EmailComponent = React.memo(({ email, onEmailChange }) => {
  console.log("Email component rendered");
  return (
    <input
      value={email}
      onChange={(e) => onEmailChange(e.target.value)}
    />
  );
});
```
