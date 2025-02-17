### 컬렉션 클래스들의 제네릭 타입이 런타임에서 어떻게 처리되는가?
컬렉션 클래스의 제네릭 타입들은 런타임 시점에 타입 이레이저에 의해 타입 정보가 제거되며 Object 또는 특정 상위 타입으로 변환됩니다.  
ex) 제네릭 타입이 Object로 변환

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

ex) 제네릭 타입이 특정 상위 타입으로 변환

    class Box<T extends Number> { // Number의 하위 타입만 가능
        private T value;
    
        public void set(T value) { this.value = value; }
        public T get() { return value; }
        }
    
    class Box {
        private Number value; // T → Number로 변환됨
    
        public void set(Number value) { this.value = value; }
        public Number get() { return value; }
    }

### 타입 이레이저(Type Erasure)란?
타입 이레이저란 제네릭이 컴파일될 때 제네릭 타입 정보가 제거되는 과정을 의미합니다. 즉, 제네릭은 컴파일 타임에만 존재하며 런타임에는 사라지는 특성을 가집니다.  

### 타입 이레이저로 제네릭 타입을 제거하는 이유는?
1. 기존 Java(JDK 1.4 이하)와 호환되도록 하기 위해(jdk 1.4 이하에서는 제네릭이 없었으므로 강제 형 변환이 필요했었습니다.)  
2. 런타임 성능 최적화(런타임에 타입 검사를 수행하면 성능이 저하 및 불피룡한 메모리 사용 증가)
즉, 타입 이레이저는 컴파일 시 타입 안정성 보장과 런타임 성능 최적화를 동시에 달성하기 위한 것입니다.

### 이를 우회하는 방법(Super Type Token)은?
기본적으로 제네릭은 런타임 시에 타입 이레이저에 의해 제네릭 정보가 제거됩니다. 하지만 슈퍼 타입 토큰을 이용해 제네릭을 런타임에도 유지할 수 있습니다.

