---
layout: posts
title: "🪐 [React] react-hook-form에 대한 얕은 고찰"
categories:
  - React
last_modified_at: 2023-01-30
author_profile: true
tags:
  - react-hook-form
  - react
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---

FilterDropdown을 만들던 중의 생각 변화의 흐름



## 개요

참고) `FilterDropdown` 컴포넌트는 input 두 개와 checkbox 하나를 가진 드롭다운 컴포넌트이다.

![filter](https://user-images.githubusercontent.com/48341341/215520739-159431e6-b5a0-429f-bdb1-db53798a8cea.png)

1. 처음에는 그냥 다 때려박아서 만들었다가
2. 진도율 input을 컴포넌트로 빼자 (invalid 피드백 등을 함께 관리하려고)
3. 이러니까 값이랑 에러를 다 props로 받아와야하네
4. 적당히 리팩토링
5. 기회 되면 input 관련 valid 체크하는거랑 다 커스텀 훅으로 제공해야지
6. 어? 그게 바로 react-hook-form..?

해서 리팩토링을 진행하게 된 것이었다.


## 본론

### 그래서,

열심히 react-hook-form으로 갈아치웠다. 

### Custom Validation with dependency

아래처럼 진도율 구간 관련한 유효성 검사 로직이 필요했다. validate라는 항목 추가로 가능했는데, 에러 메세지 넘겨주는 부분에서도 살짝 애를 먹었었다. (공식 문서를 잘 찾아보면 친절하게 나와있는데 잘 안찾아본 자의 최후..)

![filter-error](https://user-images.githubusercontent.com/48341341/215521096-210bd0a5-e6b0-4e57-989b-fc276f3e537c.png)

```jsx
// 보통은 이렇게,
{
	max: {
		value: 100,
		message: '100보다 작은 숫자를 입력해주세요.'
	}
}

// 근데 커스텀하게 주는 경우는 이렇게..!
{
	validate: (v) => (v < max) || '에러 메세지...'
}
```

### input이 변경 될 때마다 재검사가 필요한걸?

그래, 이건 해결했는데…

진도율 구간 관련 유효성 검사는 시작 구간(min)에만 추가하되, 끝 구간(max)이 변경되면 검사를 다시 해주어야 했다. (`[20, 10]`에서 `[20, 100]`으로 변경하면 invalid를 빼주어야하니까)

그래서 생각한 방법은 validate를 추가하지 않고 setErrors를 통해 min, max 값이 변경될 때마다 error를 set, reset 해주기였다.

```jsx
useEffect(() => {
		if (inputKey === MIN) {
			if (minValue > maxValue) {
				setError(MIN, {
					type: 'range',
					message: '진도율 시작 구간은 끝 구간보다 크지 않게 설정해 주세요.',
				});
			} else if (minValue < 100) {
				clearErrors(MIN);
			}
		}
	}, [minValue, maxValue]);
```

> 아 지금 생각엔 저 유효성 문구를 min, max 둘 다에게 주고 유효성 검사할 때도 v가 아니라 min, max로 했으면..?

그런데...
![hook-form-valid](https://user-images.githubusercontent.com/48341341/218104372-ce63b72c-3853-4428-a1f2-0b626a06a782.png)

validate 체크가 input이 변할 때 마다 시행 하는게 아닌가보다..?

아래처럼 validate callback 내부에 console을 ‘inner’로 찍어보았는데 매번 찍히지가 않음.

validate가 트리거 되는 시점(max: 100이면 input이 100이 되는 그 시점)에만 불리고, 확인하는 방식인 것 같음(그 외에는 유지 - 에러 메세지가 계속 보이거나, 사라진 채로)

따라서 나는 커스텀한 유효성 검사로, watch로 변경되고 있는 min 이 max 보다 작거나 같은지 확인하도록 했으니까, 처음으로 min > max 가 되는 그 시점에 한 번 불려서 에러가 찍히고, 그 이후에는 에러 메세지가 사라지는 듯?

말을 굉장히 정신 없게 써두었는데, 한 마디로 지금 내가 짜둔 코드에서는 input 값이 매번 바뀔 때마다 유효성 검사를 수행하지 않는다가 포인트. 따라서 내가 원하는 동작을 만들어내기 위해서는 어쩔 수 없이 useEffect 등으로 변경될 때마다 검사를 따로 수행해주어야 한다.


`reValidateMode: 'onChange’` 라는 걸 찾았는데, 이것도 완전히 내가 생각한대로 동작하지는 않는다.. ㅜ (코드 조각은 아래에)

```jsx
{...register(inputKey, {
						max: {
							value: 100,
							message: '100 이하의 숫자를 입력해 주세요.',
						},
						validate: {
							range: () => {
								console.log('inner', minValue, maxValue);

								return (
									minValue <= maxValue ||
									'진도율 시작 구간은 끝 구간보다 크지 않게 설정해 주세요.'
								);
							},
						},
						setValueAs: (v) => (v !== '' ? Number.parseInt(v, 10) : ''),
					})
}
```

이해 안되는 거 하나 또 발견했다.

왜 isValid 는 true 인데 errors는 남아있지? (isValid 가 맞는 상황이다.)
![hook-form-error-console](https://user-images.githubusercontent.com/48341341/218105351-bc1606cd-3522-4246-bf02-aea11a7df82e.png)

동영상 첨부를 못했는데, 상황을 정리하자면 errors가 한 박자 느리게 렌더링 되고 있었다. (다른 요소에 의해 렌더링이 한 번 더 일어나면 맞는 값..)


### triger

![trigger-issue](https://user-images.githubusercontent.com/48341341/218105633-0e769470-5b09-47b9-897f-ed0791545f68.png)
![trigger](https://user-images.githubusercontent.com/48341341/218105662-82858820-4677-4bef-8ca3-fcfd35689db8.png)

아무래도 trigger를 사용해야할 것 같지?

의존성이 있는 유효성 검사는 내가 위에서 시도했던 코드들로든 제대로 동작 안하는게 맞았다. (좀 고쳐주지.. 왜 이렇게 따로 trigger를 빼놨을까? 렌더링 퍼포먼스 때문인가?)

무튼 아래와 같이 해결했다. 요구사항에 맞추어, 범위 유효성 검사는 min에만 피드백 문구를 띄우도록 해야하므로 validate도 min일 경우에만 추가해주고, 

useEffect를 통해 max 가 변경될 때 마다 min을 trigger 해준다.

```jsx
useEffect(() => {
		trigger(MIN);
	}, [maxValue]);

...

{...register(inputKey, {
						max: {
							value: 100,
							message: '100 이하의 숫자를 입력해 주세요.',
						},
						...(inputKey === MIN && {validate: {
							range: () =>
								minValue <= maxValue ||
								'진도율 시작 구간은 끝 구간보다 크지 않게 설정해 주세요.',
						}}),
						setValueAs: (v) => (v !== '' ? Number.parseInt(v, 10) : ''),
					})}
```

### 그 외 했던 고민들

- 하나의 필드 안에 동시에 두개의 에러 메세지를 띄울 순 없나?
    - 에 대해선 아직 답을 찾지 못했다.
- 어떠한 이유에서인지 input에 입력한 값들이 따로 state로 관리되어야 한다고 생각했었음
    - 내가 적용하고 reset()을 호출하는 바람에 값이 다 엉망진창이 되어서 그랬었다…
- 적용하고, reset 하고, 각각의 input을 입력, 변경하는 과정에서 문제가 없던 부분이 동작하지 않게 되면서 기존 코드를 유지해야겠다는 판단을 내렸다.
    - 작업을 빨리 마무리해야한다는 약간의 압박감도 있었다.
    - 지금 생각해보면 내가 잘못 코딩했었다..
- 또한 당시에는, react-hook-form을 적용하게 되면서 비즈니스 요구사항을 만족시키기 위한 다른 불필요한 코드들이 더 추가되었다는 생각에 쓰지 말아야겠다는 판단도 했었다.
    - ex. react-query를 사용하기 위한 더 많은 코드들
    - 근데, 유지 보수 측면에서 모두가 동일한 방식으로 코드를 작성하고 빠르게 이해할 수 있다는 점도 간과해서는 안되겠다.
        - 요구사항을 달성하기 위한 더욱 추가적인 코드들 vs 모두가 같은 방식으로 비동기 패칭을 해온다 (다른 개발자가 유지보수 하러 왔을 때 초기 이해가 쉽다)
    - 또 하나의 생각, 꼭 라이브러리 안에서 해결할 필요는 없다?
        - 여기에 대해서는 내 생각에 아직 확신이 없긴 한데, 라이브러리를 사용하면서도 해당 라이브러리를 통해서 깔끔하게 해결할 수 없는 부분은 자체적으로 관리하는 것도 방법이라는 생각이 들었다.
        - 또 이번엔 그렇게 해결했다.

### 최종 구조
trigger를 쓰면서 마무리하게될 줄 알았으나, 해당 로직이 추가되면서 처음 설계했던 컴포넌트 구조가 굉장히 복잡하게 변해버렸다. 폼과 진도율 입력하는 인풋을 분리해서 설계했었는데, 양쪽 컴포넌트에서 유효성 검사, 상태 변화에 따른 다른 상태값 변경 등의 로직이 분산되어 버려서 각 컴포넌트의 역할이 모호해지고 어떻게 동작하는지 이해하기가 어려워진 것이다.


- react-hook-form을 사용하되, errors와 isValid는 `FilterDropdown`에서 상태로써 관리한다.
    - 현 구조에서는 값이 변경됨에 따라 여러 input들이 의존성을 가지고 얽혀있어 react-hook-form의 유효성 검사를 그대로 사용할 이점이 없다고 판단
- 이에 따라 `ProgressInput`의 역할을 defaultValue를 받아와 form에 register 하는 것으로 단순화
    - 필요한 메소드가 register 하나 뿐이라서 `useFormContext`를 쓰지 않는 편이 코드가 더 간결할 것 같아 제거

**FilterDropdown**

작성한 코드 중에서, 컨셉을 확인할 수 있도록 잘라옴
```jsx
import React, { useState, useEffect } from 'react';
import { useForm, useWatch } from 'react-hook-form';

import { checkValidation, checkIsValid } from '../../utils/filterUtils';

const ProgressInput = ({ defaultValue, inputId, register, errors = [] }) => {
	const isValid = errors.length === 0;

	return (
		<div className="w-100">
			<input
				defaultValue={defaultValue}
				id={inputId}
				{...register(inputId, {
					setValueAs: (v) => (v !== '' ? Number.parseInt(v, 10) : ''),
				})}
			/>
		</div>
	);
};

function FilterDropdown() {
	const [errors, setErrors] = useState({});
	const [isValid, setIsValid] = useState(true);

	const { register, handleSubmit, reset, setValue, control } = useForm({
		mode: 'onChange',
		defaultValues: {
			minValue: minProgress ?? 0,
			maxValue: maxProgress ?? 100,
			showMaxPercent: minProgress === 100 && maxProgress === 100,
		},
		reValidateMode: 'onChange',
	});

	const min = useWatch({ control, name: MIN });
	const max = useWatch({ control, name: MAX });
	const showMax = useWatch({ control, name: SHOW_MAX });

	useEffect(() => {
		if (showMax) {
			setValue(MIN, 100);
			setValue(MAX, 100);
		} else {
			setValue(MIN, min);
			setValue(MAX, max);
		}
	}, [showMax]);

	useEffect(() => {
		// iunput의 값이 변경될 때마다 자체적으로 구현한 유효성 검사 수행
		// 위에서도 알 수 있듯이, input 3개가 모두 깊은 의존성을 가지고 있음
		// react-hook-form에서 단일 input에 대한 유효성 검사는 용이하나, 위와 같은 상황에선 맞지 않다고 판단
		setErrors((prev) => ({
			...prev,
			[MIN]: checkValidation(min, max, errors[MIN]),
			[MAX]: checkValidation(max, max, errors[MAX]),
		}));

		if (min === 100 && max === 100) {
			setValue(SHOW_MAX, true);
		} else {
			setValue(SHOW_MAX, false);
		}
	}, [min, max]);

	useEffect(() => {
		setIsValid(checkIsValid(errors));
	}, [errors]);

	return (
		<Dropdown isOpen={isOpen} toggle={toggle}>
			<DropdownToggle>
				...
			</DropdownToggle>

			<DropdownMenu>
					<Form	id="progress-filter-form" onSubmit={handleSubmit(handleApply)}>
						<ProgressInput
							defaultValue={min}
							inputId={MIN}
							register={register}
							errors={errors[MIN]}
						/>
						<ProgressInput
							defaultValue={max}
							inputId={MAX}
							register={register}
							errors={errors[MAX]}
						/>

						<Label className="ml-3" check>
							<input
								className="form-check-input"
								name={SHOW_MAX}
								type="checkbox"
								{...register(SHOW_MAX)}
							/>
							<div className="ml-2">진도율 100%만 보기</div>
						</Label>
					</Form>
					...
			</DropdownMenu>
		</Dropdown>
	);
}

export default FilterDropdown;
```

**utils**

```jsx
import _find from 'lodash/find';
import _remove from 'lodash/remove';

/**
 * @param {*} value Number
 * @param {*} maxValue Number
 * @param {*} errorMessages Array of {type: '', message: ''}
 * @returns Array of {type: '', message: ''}
 */
const ERROR_TYPE_MIN = 'min';
const ERROR_TYPE_MAX = 'max';
const ERROR_TYPE_RANGE = 'range';
export const checkValidation = (value, maxValue, errors = []) => {
	const currentErrors = errors;

	const hasMinError = _find(currentErrors, { type: ERROR_TYPE_MIN });
	if (value < 0) {
		!hasMinError &&
			currentErrors.push({
				type: ERROR_TYPE_MIN,
				message: '0 이상의 숫자를 입력해주세요.',
			});
	} else {
		_remove(currentErrors, { type: ERROR_TYPE_MIN });
	}

	const hasMaxError = _find(currentErrors, { type: ERROR_TYPE_MAX });
	if (value > 100) {
		!hasMaxError &&
			currentErrors.push({
				type: ERROR_TYPE_MAX,
				message: '100 이하의 숫자를 입력해 주세요.',
			});
	} else {
		_remove(currentErrors, { type: ERROR_TYPE_MAX });
	}
	if (value > maxValue) {
		const hasRangeError = _find(currentErrors, { type: ERROR_TYPE_RANGE });
		!hasRangeError &&
			currentErrors.push({
				type: ERROR_TYPE_RANGE,
				message: '진도율 시작 구간은 끝 구간보다 크지 않게 설정해 주세요.',
			});
	} else {
		_remove(currentErrors, { type: ERROR_TYPE_RANGE });
	}

	return currentErrors;
};

/**
 * @param {*} errors Object
 * 	{
 * 		min: [ {type: '', message: ''}, ... ],
 * 		max: [ {type: '', message: ''}, ... ],
 *  	...
 *  }
 * @returns Boolean
 */
export const checkIsValid = (errors = {}) => {
	const allErrors = Object.values(errors);
	const hasError =
		allErrors.filter((eachErrors) => eachErrors.length > 0).length > 0;

	return !hasError;
};
```

**얻은 이점**

- `setValueAs: (v) => (v !== '' ? Number.parseInt(v, 10) : '')` 를 통해 입력값이 숫자임을 보장할 수 있게 되면서 input value이 string일 경우도 고려하기 위해 추가된 코드들을 삭제할 수 있었다.

## 결론

새로운 라이브러리를 처음 사용할 때는 실제로 적용된 코드를 보며 사용법을 빠르게 익히는 것도 중요하지만, 공식 문서를 정독하는 시간이 꼭 필요하다.
또, 꼭 라이브러리를 통해서만 모든 것을 해결하려고 하지 말자. 그저 하나의 편리한 도구일 뿐이다.

- 지금 나한테 완전히 필요한 도구가 아니라면, 필요한 부분만 사용하고 없는 부분은 내가 만들어서 사용할 수 있다는 생각

**그리고, 항상 생각해야 할 것**

- 이 라이브러리는 어떤 문제를 해결하기 위해 나온 것인가?
- 지금 내가 해결하고자 하는 문제에 대한 적합한 해결책인가?

위 질문을 꼭 먼저 해보도록 하자. 이제는 단순히 `빠르게 동작하는 코드를 만들어 내는 것` 에 그치면 안되고, `유지 보수가 쉬운 코드, 적게 짜서 해결할 수 있는 좋은 구조의 코드`를 생각해내야 한다.