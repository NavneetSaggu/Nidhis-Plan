# Composition Over Inheritance

- Inheritance tells you what a class is, not a good way to extend what a class does.
- Composition can help you extend what a class does, in a much better way.

# Inheritance

```java
interface Piece
{
		 boolean canMove();
}

class Rook implements BasePiece
{
	   @Override
		 boolean canMove() {
				// Check for vertical
				// Check for horizontal
		 }
}

class Queen implements BasePiece
{
	   @Override
		 boolean canMove() {
				// Check for vertical
				// Check for diagonal
				// Check for horizontal
		 }
}

class Bishop implements BasePiece
{
	   @Override
		 boolean canMove() {
				// Check for diagonal
		 }
}
```

## Cons

If diagonal behaviour / any other kind of move behaviour changes, you will have to make changes in all the classes that implement this behaviour.

This is not good, why should Bishop know how to do diagonal moves? It only needs to know that it can do diagonal moves, not the implementations of it.

If a new move is added, you might have to make changes to all pieces.

# Composition

```java
interface Move
{
	    boolean canMove();
}

class DiagonalMove implements Move
{
	    @Override
			boolean canMove() {
				// Check for diagonal
		 }
}

class HorizaontalMove implements Move
{
	    @Override
			boolean canMove() {
				// Check for horizontal
		 }
}

class Piece
{
	    List<Move> moves;
			public Piece(List<Move> moves)
			public canMove();
 }
```

Now piece can simply check whether a particular move is possible or not

## Pros

Piece is agnostic of what kind of moves it has. This follows single responsibility principle better.