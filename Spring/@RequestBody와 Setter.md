# `@RequestBody`와 Setter

본 주제에 대해 이야기하기 전에 먼저 `@ModelAttribute`에 대해 이야기해보려합니다.

<br>

## `@ModelAttribute`
우리는 `Spring`에서 `Reqeust Parameter`를 얻기 위해 `@ModelAttribute`를 사용하곤합니다.

값을 바인딩하여 우리가 원하는 객체로 변환해주는 역할을 하는데

![image](https://user-images.githubusercontent.com/60773356/142987672-4269fee3-142e-4b5a-a72a-2fefa975f3e7.png)

아래와 같이 형식에 맞춰 값이 넘어오면 원하는 객체로 손쉽게 변환할 수 있다는 큰 장점이 있습니다.
> 물론 타입이나 형식이 안맞으면 `TypeConverter`에서 예외가 발생함

![image](https://user-images.githubusercontent.com/60773356/142987966-9ac7c68d-4c57-4ad3-8b8a-a8024901411f.png)

값을 바인딩하길 원하는 객체인 `RequestDto`는 `name`과 `age`필드 두가지를 가지고 있는데

실제 요청을 postman을 사용해 `html-form` 형식으로 전달해보겠습니다.(`query parameter`도 결과는 동일하다.)

![image](https://user-images.githubusercontent.com/60773356/142988286-22605592-a045-4e9e-9b8f-7b374aa53fb5.png)
![image](https://user-images.githubusercontent.com/60773356/142988299-3d42a317-2886-4448-ba67-4985fb32590c.png)

디버거를 통해 `breakpoint`를 찍어보면 값이 전혀 찍히지 않았고 당연히 로그에도 `null`이 출력되는 것을 확인할 수 있습니다.

사실, `Spring`을 좀 써본 사람이라면 문제가 무엇인지 다 알고 있을 것입니다.

<br>

#### `@ModelAttribute`는 값을 객체로 바인딩할 때 프로퍼티 접근법을 사용!!

1. 해당 객체를 생성(기본 생성자)
2. **프로퍼티 접근법인 `setter`를 사용하여 넘어온 값을 객체에 주입**

보이진 않지만 `Spring MVC`는 아래와 같은 동작을 하고 있을 것입니다.
```java
RequestDto dto = new RequestDto();
dto.setName(값);
dto.setAge(깂);
```

하지만 위의 `RequestDto`에는 기본 생성자는 존재하지만 `setter`가 존재하지않으므로 값을 바인딩하지 못해 `null`이 반환된 것이죠.

<br>

 ![image](https://user-images.githubusercontent.com/60773356/142989257-d7c8c28b-aa18-4de3-8e17-31b983715131.png)

`setter`만 추가하주면 정상적으로 값이 바인딩됩니다.(기본 생성자는 자바가 알아서 만들어줌)

![image](https://user-images.githubusercontent.com/60773356/142989393-5e5808a6-b366-4a95-a194-5bf0670bffc5.png)

<br>

하지만, `@Setter`를 사용하는 것이 뭔가 꺼려지고 그러면 안될 것 같은 기분이 들 수 있는데(~~사실 나임~~)

그럴 땐, 모든 필드를 매개변수로 받는 생성자를 만들면됩니다..(`@AllArgsConstructor`)

저는 정적 팩토리 메서드를 사용하여 이를 만들어봤는데 `setter`가 없더라도 정상적으로 값이 바인딩되는 것을 확인할 수 있었습니다.

![image](https://user-images.githubusercontent.com/60773356/142989963-4aaf4e1b-1f2b-4e2f-91b2-491d1becc668.png)

아마 `setter`가 존재하지않을때 요청을 통해 넘어온 값을 바로 넣어 `Spring MVC`가 생성해주는 것이 아닐까 생각됩니다.

```java
RequestDto dto = new RequestDto(name, age);
```

**이렇게!**

<br>

## `@ModelAttribute` 추가
어제 위 부분에 대한 궁금증으로 대략 2~3시간 동안 디버거 모드로 찍어보고 `Spring MVC`의 내부 코드를 뜯어보며 분석을 좀 해봤습니다.

뜯다보니 오랜만에 `ArgumentResolver`도 보이고해서 전체적인 개념을 다시 한번 잡을 수 있었는데

제가 알게된 점은 다음과 같습니다.

`@ModelAttribute`를 처리하기 위해서는 `ModelAttributeMethodProcessor`라는 `ArgumentResolver`를 사용합니다.

![image](https://user-images.githubusercontent.com/60773356/143193437-3f49af15-eaff-43f7-911b-21597854450e.png)

크게 역할은 값을 바인딩하는 역할과 값을 검증(`bindingResult`)하는 역할을 합니다.

내부를 살펴보니 가장 핵심 메소드는 `createAttribute`와 `constructAttribute`였습니다.

### createAttribute
![image](https://user-images.githubusercontent.com/60773356/143194171-67b47eb8-ca8e-48bf-bab0-990846e1d04c.png)

`javaDoc`에 잘 설명이되어 있는데 기본적으로 기본 생성자인 `NoArgsConstructor`를 사용하지만 적절한 생성자가 존재한다면 그것을 통해 객체를 생성합니다.

그 후, setter를 통해 값을 주입하는 방법을 사용하는 것이죠.

### constructAttribute
![image](https://user-images.githubusercontent.com/60773356/143194733-0f88d591-7fc0-46b0-9db5-1743af1c505d.png)

<details markdown="1">
<summary>접기/펼치기</summary>

```java
	protected Object constructAttribute(Constructor<?> ctor, String attributeName, MethodParameter parameter,
			WebDataBinderFactory binderFactory, NativeWebRequest webRequest) throws Exception {

		if (ctor.getParameterCount() == 0) {
			// A single default constructor -> clearly a standard JavaBeans arrangement.
			return BeanUtils.instantiateClass(ctor);
		}

		// A single data class constructor -> resolve constructor arguments from request parameters.
		String[] paramNames = BeanUtils.getParameterNames(ctor);
		Class<?>[] paramTypes = ctor.getParameterTypes();
		Object[] args = new Object[paramTypes.length];
		WebDataBinder binder = binderFactory.createBinder(webRequest, null, attributeName);
		String fieldDefaultPrefix = binder.getFieldDefaultPrefix();
		String fieldMarkerPrefix = binder.getFieldMarkerPrefix();
		boolean bindingFailure = false;
		Set<String> failedParams = new HashSet<>(4);

		for (int i = 0; i < paramNames.length; i++) {
			String paramName = paramNames[i];
			Class<?> paramType = paramTypes[i];
			Object value = webRequest.getParameterValues(paramName);

			// Since WebRequest#getParameter exposes a single-value parameter as an array
			// with a single element, we unwrap the single value in such cases, analogous
			// to WebExchangeDataBinder.addBindValue(Map<String, Object>, String, List<?>).
			if (ObjectUtils.isArray(value) && Array.getLength(value) == 1) {
				value = Array.get(value, 0);
			}

			if (value == null) {
				if (fieldDefaultPrefix != null) {
					value = webRequest.getParameter(fieldDefaultPrefix + paramName);
				}
				if (value == null) {
					if (fieldMarkerPrefix != null && webRequest.getParameter(fieldMarkerPrefix + paramName) != null) {
						value = binder.getEmptyValue(paramType);
					}
					else {
						value = resolveConstructorArgument(paramName, paramType, webRequest);
					}
				}
			}

			try {
				MethodParameter methodParam = new FieldAwareConstructorParameter(ctor, i, paramName);
				if (value == null && methodParam.isOptional()) {
					args[i] = (methodParam.getParameterType() == Optional.class ? Optional.empty() : null);
				}
				else {
					args[i] = binder.convertIfNecessary(value, paramType, methodParam);
				}
			}
			catch (TypeMismatchException ex) {
				ex.initPropertyName(paramName);
				args[i] = null;
				failedParams.add(paramName);
				binder.getBindingResult().recordFieldValue(paramName, paramType, value);
				binder.getBindingErrorProcessor().processPropertyAccessException(ex, binder.getBindingResult());
				bindingFailure = true;
			}
		}

		if (bindingFailure) {
			BindingResult result = binder.getBindingResult();
			for (int i = 0; i < paramNames.length; i++) {
				String paramName = paramNames[i];
				if (!failedParams.contains(paramName)) {
					Object value = args[i];
					result.recordFieldValue(paramName, paramTypes[i], value);
					validateValueIfApplicable(binder, parameter, ctor.getDeclaringClass(), paramName, value);
				}
			}
			if (!parameter.isOptional()) {
				try {
					Object target = BeanUtils.instantiateClass(ctor, args);
					throw new BindException(result) {
						@Override
						public Object getTarget() {
							return target;
						}
					};
				}
				catch (BeanInstantiationException ex) {
					// swallow and proceed without target instance
				}
			}
			throw new BindException(result);
		}

		return BeanUtils.instantiateClass(ctor, args);
	}
```

</details>

`createAttribute`가 적절한 생성자를 찾고 `constructAttribute`를 통해 해당 생성자로 새로운 객체 인스턴스를 생성하는 구조입니다.

적절한 생성자를 찾기 때문에 매우 다양한 조합이 가능해집니다.

```java
@Getter
@Setter
public class RequestDto() {
  private String name;
  private Long age;
 
  public RequestDto(String name) {
    this.name = name;
  }
}
```

위와 같은 경우에는 `name`을 받는 생성자를 통해 객체를 생성하고 `setAge`를 통해 `age`값을 바인딩할 것입니다.

물론, 위에서 언급했듯이 `AllArgsConstructor`또한 정상적으로 값을 바인딩할 수 있습니다!

#### 결론은 적절한 생성자를 먼저 찾고 그 뒤에 바인딩되지 않은 값을 `setter`를 통해 바인딩해주는 순서로 `@ModelAttribute`는 동작합니다.



<br>
<br>

--------

## `@RequestBody`
그럼 원래 주제로 돌아가 `@RequestBody`는 값을 어떻게 바인딩할까??

`@ModelAttribute`와 값을 바인딩한다는 관점에서는 동일하지만 이는 `HTTP Message Body`를 읽는다는 점에서 다릅니다.

대체로 `JSON`을 통해 `REST API`로 애플리케이션을 구성하게 된다면 가장 많이 쓰게될 것인데,

이는 `@ModelAttribute`처럼 생략을 할 수 없습니다.

`setter`를 없애고 `JSON`으로 요청을 전달해보겠습니다.

![image](https://user-images.githubusercontent.com/60773356/142990700-96fd06aa-0de3-4e5e-a400-762b0c9848d0.png)

![image](https://user-images.githubusercontent.com/60773356/142990715-94efd27a-002c-4040-9fcb-115d9a3419ef.png)

`@RequestBody`용 컨트롤러를 하나 생성하고 `ReqeustDto`는 필드와 `getter`만 남긴 채 모두 주석처리하였습니다.

```
{
    "name": "민철",
    "age": "25"
}
```

`JSON` 형식으로 `Body`에 담아 요청을 전송해보겠습니다.

![image](https://user-images.githubusercontent.com/60773356/142992106-a9446a42-0f63-4189-aeb5-700e59ccdfda.png)

어랍쇼??😳

<br>

맞습니다. **`@RequestBody`를 사용하게되면 `setter`는 전혀 필요없습니다!**

<br>


그 이유는 간단합니다.

`HTTP Message Body`를 읽기 위해서 `Spring`은 `HTTP Message Converter`를 사용합니다.

`HTTP Message Converter`를 구현한 `Converter`중에 넘어온 값을 읽을 수 있는 `Converter`를 찾습니다.

![image](https://user-images.githubusercontent.com/60773356/142994375-d08bfbeb-914a-4f8c-850f-3270d519115a.png)

읽을 수 있는 `Converter`가 존재한다면 `read`메서드를 사용하여 값을 읽습니다.

![image](https://user-images.githubusercontent.com/60773356/142994510-6b493b69-808b-4091-b742-48381e55e7b8.png)

앞선 예시에서 저는 `JSON`형식으로 값을 전달하였는데, 여기에는 **`Jackson2HttpMessageConverter` 가 사용**됩니다.

그리고 **`JSON`과 `Java`객체와의 변환은 `ObjectMapper`를 사용하는데, `Jackson2HttpMessageConverter`에서도 마찬가지**입니다.

그러니 `setter`는 전혀 필요하지 않는 것입니다.

<br>

![image](https://user-images.githubusercontent.com/60773356/142994925-f1ebe2a6-6ad0-47e0-92f1-b17e42ba0373.png)

`Jackson2HttpMessageConverter`의 내부 구현을 살펴보면 `read()`메서드가 존재하고 그 안에서 `readJavaType`을 호출합니다.

![image](https://user-images.githubusercontent.com/60773356/142995070-123951a7-e274-417b-a22e-b384185f11f9.png)

그리고 호출된 `readJavaType`메서드는 `ObjectMapper`를 사용하여 값을 읽는 것을 확인할 수 있습니다!








