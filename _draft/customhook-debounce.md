### 검색바에는 debounce를 걸어줘야지!

프론트 작업을 하다보면 늘 느끼는 생각이지만 실생활에서 너무 당연하게 간단히 사용하는 기능들을 실제로 구현할 때는 녹록지 않다는 것이다. 특히 이전에 채팅을 개발할 때와 이번에 검색바 구현에서 많이 느꼈다. 
역시 기본기가 탄탄해야 한다는 것을 말하는 것일까..?

무튼 검색바를 구현하며 focusIn, Out을 비롯해 DB에서 find 결과를 한 번에 가져온 뒤 클라이언트 단에서 매칭 우선순위에 따라 정렬하는 등 여러가지를 고려할 수 있었다.
그 중에서도 이번엔 디바운스에 대해 몇 글자 적어보도록 하겠다.

// 디바운스에 대한 정의, 설명

// lodash를 사용할 수 있다는 것을 모르고 커스텀 훅을 통해 직접 구현해 보앗다..

// 얼마 전에 커스텀 훅을 사용해보아서 복습하는 기분이고 좋았다 ^.^

```
import React, { useState, useEffect } from 'react';

function useDebounce(value, delay) {
	const [debouncedValue, setDebouncedValue] = useState(value);
	
	useEffect(() => {
		const timer = setTimeout(() => {
			setDebouncedValue(value);
		}, delay);
		
		return () => {
			clearTimeout(timer);
		}
	}, [value]);
	
	return debouncedValue;
}

export default useDebounce;


---
import useDebounce from 'hooks/useDebounce';


useEffect(() => {
		console.log(debouncedValue);
		const res = axios.get('/api/organization/search', {
			params: {
				searchInput: input,
			}
		});
	}, [debouncedValue]);
	
```
