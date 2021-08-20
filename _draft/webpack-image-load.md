npm install file-loader 하고

webpack.config.js에

			{
    			test: /\.(gif|svg|jpg|png)$/,
				use: [
					{
						loader: "file-loader",
					}
				]
    			
  			},

이거 추가

> https://13akstjq.github.io/react/2019/10/08/webpack-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-png%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0.html