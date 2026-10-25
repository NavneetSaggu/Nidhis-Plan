# Builder

```java
class Order {
	int amount;
	int quantity;
	String seller;
	String buyer;
	Item item;
	public Order(int amount, int quantity, string seller, string buyer, Item item) {
		this.amount = amount;
		this.item = item;
		this.buyer = buyer;
		this.seller = amsellerount;
		this.quantity = quantity;
	}
}

class OrderBuilder() {
	int amount;
	int quantity;
	String seller;
	String buyer;
	Item item;
	
	public OrderBuilder setAmount(int amount) {
		this.amount = amount;
		return this;
	}
	
	public OrderBuilder setQuantity(int quantity) {
		this.quantity = quantity;
		return this;
	}
	
	public OrderBuilder setSeller(String seller) {
		this.seller = seller;
		return this;
	}
	
	public OrderBuilder setBuyer(String buyer) {
		this.buyer = buyer;
		return this;
	}
	
	public OrderBuilder setItem(Item item) {
		this.item = item;
		return this;
	}
	
	public Order build() {
		return new Order(
		  this.amount,
      this.quantity,
      this.seller,
      this.buyer,
      this.item
		);
	}
}

```