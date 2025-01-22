## String Pool이란?
String Pool은 Java에서 문자열 리터럴을 효율적으로 관리하기 위해 JVM의 Heap 메모리 내에 별도로 마련된 공간입니다.
이 공간은 동일한 문자열 리터럴을 중복 저장하지 않고, 하나의 인스턴스를 재사용하여 메모리 사용량을 최적화합니다.

### String Pool의 주요 특징
#### 문자열 리터럴 공유 
동일한 문자열 리터럴은 String Pool에 한 번만 저장되고, 그 이후 동일한 문자열을 참조할 경우 기존 문자열 객체를 재사용합니다.
ex)
  String str1 = "madplay";
  String str2 = "madplay"; // str1과 같은 객체를 참조

#### Heap 메모리 내 위치
String Pool은 JVM의 Heap 메모리 영역에 위치합니다.

#### 불변(Immutable) 특성
문자열은 불변(Immutable)이므로 String Pool에 저장된 문자열은 수정되지 않습니다.

## 메모리에 어떻게 저장될까?
String str1 = "madplay";  //"madplay" 라는 문자열 리터럴이 String Pool에 저장될 것이고 str1은 String Pool의 "madplay"를 참조합니다.  
String str2 = "madplay";  //"madplay"는 이미 String Pool에 존재하므로, 새로운 객체를 생성하지 않고 String Pool의 "madplay"를 참조합니다.  
String str3 = new String("madplay");  //new 키워드를 사용했기 때문에 Heap 메모리에 새로운 String 객체가 생성됩니다.  
String str4 = new String("madplay");  //new 키워드를 사용했기 때문에 Heap 메모리에 새로운 String 객체가 생성됩니다.  
str1 = str2; //str1은 단순히 str2가 참조하고 있는 String Pool의 "madplay"를 참조합니다.  

str3, str4는 힙메모리에 새로운 객체를 생성하고 이 객체의 값은 String Pool의 "madplay"를 참조합니다.

String str5 = new String("madplay1") //이렇게 했을 경우 "madplay1" 문자열 리터럴이 String Pool에 저장되고 별도의 String 객체도 생성됩니다. 그리고 String 객체는 String Pool에 있는 "madplay1" 값을 참조합니다.

결론적으로는 문자열 리터럴을 생성하면 String Pool에 모두 저장됩니다.
