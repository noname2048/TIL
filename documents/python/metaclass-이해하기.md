# metaclass-이해하기
```
class Vehicle:
    max_paseenger = None
    km_per_liter = None
    
    def start(self) -> None:
        print("engine start!")
        
    def onboard(self, passenger: int) -> bool:
        if passenger < self.max_passenger:
            return True
        return False
    
    def move(self, distance_km: float) -> bool:
        if self.fuel > distance_km / self.km_per_liter:
            self.fuel -= distance_km / self.km_per_liter
            print(f"reamin fuel: {self.fuel}.")
            return True
        return False

class Car(Vehicle):
    max_paseenger = 4
    tire = 4
    
    def start(self):
        super().start()
        print("rpm -> 1000x")
        
    def refuel(self, fuel):
        self.fuel += fuel

### 요렇게 바꾸면

def start(self):
    super(Vehicle, self).start
    print("rpm -> 1000x")
        
def refuel(self, fuel):
    self.fuel += fuel

    
Car2 = type("Car", (Vehicle,), {
    "max_paseenger": 4,
    "tire": 4,
    "fuel": 6,
    "start": start,
    "refuel": refuel,
})
```
```
c = Car()
c.start()
```
```
c2 = Car2()
c2.start()
```
```
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-87-ddbe0c62aa8e> in <module>
      1 c2 = Car2()
----> 2 c2.start()

<ipython-input-77-20908ae087f9> in start(self)
     32 
     33 def start(self):
---> 34     super(Vehicle, self).start
     35     print("rpm -> 1000x")
     36 

AttributeError: 'super' object has no attribute 'start'
```
해당 느낌으로 실행했는데 잘 안된다. typeclass 가 class를 동적으로 생성시키는 거라면, super에 해당하는 것은 무얼까?
