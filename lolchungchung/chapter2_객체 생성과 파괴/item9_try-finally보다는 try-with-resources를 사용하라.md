Item9 : try-finally 보다는 try-with-resoucres를 사용하라
목적 : 올바른 자원닫기를 보장하는 수단을 알아보자

Item8에 이어서 ...
그래서 자원닫기를 보장하는 수단은 어떤것을 사용해야 하는가

예시코드 : 전통적인 자원닫기 try-finally
static String firstLineOFfile(String path) throws IOException {
	BufferedReader br = new BufferedReader( new FileRedaer(path));
	try {
		return br.readLine();
	}
	finally{
		br.close();
	}
}

그러나 전통적인 방법에는 예외 발생시 try문과 finally문에서 모두 발생하여 
finally 문 예외가 첫번째 예외를 삼켜버려서 첫번째 예외에 대해서는 추적이 불가능하다.

=> try-with-resources를 사용한다. (이 구조를 사용하려면 AutoCloseable 인터페이스를 구현해야함.

예시코드 : try-with-resources
static String firstLineOfFile(String path, String defaultVal){
	try(BufferedReader br = new BufferedReader(new FileReader(path))){
		return br.readLine();
	}
	catch(IOExeception e){
		return defaultVal;
	}
}
