### 이해한 것
- context를 객체로 전달하는 것은 주의해야 할 일이다.
 - context는 redux와 달리 렌더링 최적화가 되어 있지 않다.
 - 따라서
 - 1) useMemo를 통해 value를 전달하려면 관련이 있는 정보들만 묶어서 전달하거나
 - 2) context 자체를 관련 있는 값들과 연결시켜 쪼개서 value의 전달이 관련 있는 것들만 자식들에게 전파되도록 해야 한다.
 - 그렇다면 객체를 context에 사용하려면 많은 관심이 필요한데, redux나 zustand를 쓰는 게 나을 수도 있지만 성능상 큰 이슈가 없다면
   사용자는 리렌더링을 알아채지 못하므로, 장단점을 고려하자

   ```javascript
interface Props {
  children: ReactNode;
  params?: LogPayloadParameters;
}

const LogParamsContext = createContext<LogPayloadParameters | null>(null);

export function LogParamsProvider({ children, params }: Props) {
  return (
    <LogParamsContext.Provider
      value={params}
    >
      {children}
    </LogParamsContext.Provider>
  );
}

export function useLogParams() {
  return useContext(LogParamsContext);
}


export function LogScreen({ children, params }: Props) {
  const router = useRouter();
  const logger = useLogger();

  useEffect(() => {
    if (router.isReady) {
      logger.screen({ params });
    }
  }, [router.isReady]);

  return <LogParamsProvider params={params}>{children}</LogParamsProvider>
}

function RegisterCarPage() {
  // ...
  return (
    <LogScreen params={{ title: PAGE_TITLE, userId }}>   
      // ...
      <RegisterCarForm onSubmit={...}/> 
      <LogClick params={{ button: '다음' }}>
        <Button onClick={...}>
          다음  
        </Button>   
      </LogClick>  
    </LogScreen> 
  );
}

function RegisterCarForm({ onSubmit }) { 
  return (  
    <form onSubmit={onSubmit}>   
      <CardNumberField />
      <LogClick params={{ button: '차량 정보 초기화' }}>
        <Button>
          차량 정보 초기화
        </Button>
      </LogClick>
      // ... 
    </form>  
  );
}


```

이 예시 역시도 RegisterCarPage가 state를 가지고 있어서 이 값이 바뀐다면 Provider의 리렌더링 및 Context가 재평가되어
해당 컴포넌트도 리렌더링된다.