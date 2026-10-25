# Mockito

Suppose the method you want to test, makes an HTTP request to some upstream service or DB to run and return it’s result.

Suppose your service client makes this call, you can’t make an object of your service client in you test method and have it make HTTP calls.

This is where you’ll have to mock your client object.

```java
UpstreaService service = Mockito.mock(UpstreaService.class);
```

You now have a mock object to use.

But you must make sure that you tell it what to return when certain methods of this objects that it is mocking are called.

These methods can be called directly from your test method, or it could be that they are being used by other objects that you’ve initialized in your test method.

You just tell Mockito, that when abc method is called with pqr value, return xyz.

```java
// "request" has to be an object that you know that calls will be made for when your test is run
when(service.callUpstreamService("request")).theReturn("response");

// you can also verify that your mock service method was called in an operation
verify(service).callUpstreamService("request");
```

# Power Mock

We can mock static, private, and final methods using PowerMock, which we can’t do with Mockito.

Suppose we’re testing an OrderService, which internally uses a NotificationUtil class.

Put the following annotations on top of your tests class

```java
@Data
public class OrderRequest {
	private int quantity;
	private String item;
	private int price;
	private String emailId;
}
```

```java
@Data public class OrderResponse {
	private int price;
	private String message;
}
```

```java
@RunsWith(PowerMockRunner.class)
@PrepareTest(Fully.Qualified.Package.Name.For.Which.Youre.Injecting.Mocks.*)
public class OrderAppTests {
	
	 @InjectMocks
	private OrderService orderService;
	OrderRequest serviceRequest = new OrderRequest(1, "Mobile", 10000, "test@gmail.com");

	@Before
	public void init() {
		MockitoAnnotations.initMocks(NotificationUtil.class);
	}
}
```

## Mocking Static Methods

Suppose NotificationUtil class has a sendEmail() static method which is used to send email in checkoutOrder() method of OrderService.

Remember, tests follow [Behavioral](_Design%20Patterns.md) programming pattern.

```java
@Test
public void testStaticMethod() {
	// given
	Strinfg emailId = "test@gmail.com";
	PowerMockito.mockStatic(NotificationUtil.class)
	// when
	when(NotificationUtil.sendEmail(emailId)).thenReturn("Success");
	// then
	OrderResponse serviceResponse = orderService.checkoutOrder(serviceRequest);
	asseertEquals("success", serviceResponse.getMessage());
}
```

## Mocking Private Methods

Suppose OrderService has a private method, getDiscount(), which we add to the flow of checkoutOrder() to get integer specifying out discount on the order.

We then subtract this discount from the price in the final order.

```java
@Test
public void testPrivateMethod() {
	// when
	OrderService spy = PowerMockito.spy(orderService);
	doReturn(1000).when(spy, "getDiscount", ArgumentMatchers.any());
	// ArgumentMatchers.any() will make PowerMock return the given fixed integer. no matter what request objects are sent to getDiscount
	// then
	OrderResponse serviceResponse = spy.checkoutOrder(serviceRequest);
	int price = serviceResponse.getPrice();
	assertEquals(9000, price);
}
```