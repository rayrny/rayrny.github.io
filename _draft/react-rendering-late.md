	
    렌더링 한 박자 씩 느릴 때 해결 방법
    
    
    /*useEffect(() => {
		// setState가 비동기라 렌더링이 한박자 느림 -> component의 경우 콜백 함수를 넣으면 되고
		// 지금처럼 함수형일 경우 useEffect를 사용하면 된다.
		console.log('[useEffect]' + fileContent);
	}, [fileContent]);*/