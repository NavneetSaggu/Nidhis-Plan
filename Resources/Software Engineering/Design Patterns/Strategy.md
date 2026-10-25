# Strategy

Suppose there’s X different ways to perform some function in your class, and there’s Y total kinds of sub-classes that there can be.

Now if X < Y, you’re going to be repeating code in some of your classes.

Instead, make X strategies, and give one to each Y of you classes.

**Example**

```kotlin
class WaterHeater {
	fun heatWater();
	fun drawPower();
}

class GasHeater: WaterHeater {
	fun heatWater() {
		// make very hot // ========= SEE HERE
		// do it quickly //
		// HOT
		// QUICK
	}
	fun drawPower() {
		// gas from pipeline
	}
} 

class OilHeater: WaterHeater {
	fun heatWater() {
		// make very hot // ========= SEE HERE
		// do it quickly //
		// HOT
		// QUICK
	}
	fun drawPower() {
		// oil from cistern
	}
}

class SolarHeater: WaterHeater {
	fun heatWater() {
		// make less hot // ========= SEE HERE
		// do it slowly //
		// LUKWARM
		// SLOW
	}
	fun drawPower() {
		// heat from sun
	}
}

class ElectricHeater: WaterHeater {
	fun heatWater() {
		// make less hot // ========= SEE HERE
		// do it slowly //
		// LUKWARM
		// SLOW
	}
	fun drawPower() {
		// electricity 
	}
}
```

The functionality for heat water is repeated inside both Electric & Solar heater, and Oil & Gas heater. 

To avoid this repeating code, we make a strategy for heating water and give it to each kind of object.

See how it avoids duplication? There ya go

```kotlin
class WaterHeater(	
	private val HeatingStrategy heatingStrategy) {

	WaterHeater(HeatingStrategy heatingStrategy) {
		this.heatingStrategy = heatingStrategy;
	}
	fun heatWater() {
		heatingStrategy.heatWater();
	}
	fun drawPower();
}

interface HeatingStrategy {
	fun heatWater()
}

class QuickHotHeatingStrategy: HeatingStrategy {
	fun heatWater() {
		// make very hot // ========= SEE HERE
		// do it quickly //
		// HOT
		// QUICK
	}
}

class SlowLukewarmHeatingStrategy: HeatingStrategy {
	fun heatWater() {
		// make less hot // ========= SEE HERE
		// do it slowly //
		// LUKWARM
		// SLOW
	}
}

class GasHeater: WaterHeater {
	GasHeater() {
		super(new QuickHotHeatingStrategy())
	}
	fun heatWater() {
		heatingStrategy.heatWater();
	}
	fun drawPower() {
		// gas from pipeline
	}
} 

class OilHeater: WaterHeater {
	OilHeater() {
		super(new QuickHotHeatingStrategy())
	}
	fun heatWater() {
		heatingStrategy.heatWater();
	}
	fun drawPower() {
		// oil from cistern
	}
}

class SolarHeater: WaterHeater {
	SolarHeater() {
		super(new SlowLukewarmHeatingStrategy())
	}
	fun heatWater() {
		heatingStrategy.heatWater();
	}
	fun drawPower() {
		// heat from sun
	}
}

class ElectricHeater: WaterHeater {
	ElectricHeater() {
		super(new SlowLukewarmHeatingStrategy())
	}
	fun heatWater() {
		heatingStrategy.heatWater();
	}
	fun drawPower() {
		// electricity 
	}
}
```