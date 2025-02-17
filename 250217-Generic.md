[### 컬렉션 클래스들의 제네릭 타입이 런타임에서 어떻게 처리되는가?
컬렉션 클래스의 제네릭 타입들은 런타임 시점에 타입 이레이저에 의해 타입 정보가 제거되며 Object 또는 특정 상위 타입으로 변환됩니다.
예를 들어 

    class Box<T> {
        private T value;
    
        public void set(T value) { this.value = value; }
        public T get() { return value; }
    }


    class Box {
        private Object value; // T → Object로 변환됨
    
        public void set(Object value) { this.value = value; }
        public Object get() { return value; }
    }

### 제네릭 타입이 컴파일 시점과 런타임 시점에서 어떻게 변하는가?


### 타입 정보가 사라지는(Type Erasure) 이유와 이를 우회하는 방법(Super Type Token)은?

### 런타임에서는 제네릭이 Object로 변환되는가, 특정 타입으로 변환되는가?
](https://bjango.com/mac/istatmenus/)
