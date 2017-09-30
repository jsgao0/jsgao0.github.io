---
title: 硬邦邦的設計原則 - SOLID
tags: 
    - Design Principle
    - SOLID
    - SRP
    - OCP
    - LSP
    - ISP
    - DIP
date: 2017-09-30 12:33:00
---

## Single Responsibility Principle - 單一職責原則
> A class should have only one reason to change.

從不同面想去理解，可以想像成每個類別存在的目的都只會有一個。

生活中的範例：
- 電風扇的功能為吹出風。
- 電腦螢幕的功能為顯示畫面。
- 書本的功能為提供資訊。

## Open Close Principle 開閉原則
> Software entities like classes, modules and functions should be open for extension but closed for modifications.

嗯... 就是盡量不要改既有的程式碼，而是盡量用延伸的方式去完成需求修改。

## Liskov's Substitution Principle - Liskov替換原則
> Derived types must be completely substitutable for their base types.

假定有個類別為`Ship`作為基底，然後有兩個類別`WarShip`和`CargoShip`繼承`Ship`。 `Ship`可以做為替換`WarShip`和`CargoShip`的容器，並且保留繼承的所有功能。

- Ship.java
``` Java
public class Ship {
    public void steer(Degree degree) {
        // Logic of steer method.
    }
    public void move(Movement movement) {
        // Logic of move method.
    }
    public void speed(Speed speed) {
        // Logic of speed method.
    }
}
```

- WarShip.java
``` Java
public class WarShip extends Ship {
}
```

- CargoShip.java
``` Java
public class CargoShip extends Ship {
}
```

- World.java
``` Java
public class World {
    public static void main(String[] args) {
        Ship warShip = new WarShip();
        Ship cargoShip = new CargoShip();
        warShip.steer(new Degree(70));
        warShip.speed(new Speed(90, 'KM/HR'));
        warShip.move(new Movement(5100, 'KM'));
        cargoShip.steer(new Degree(-124.5));
        cargoShip.speed(new Speed(20, 'KM/HR'));
        cargoShip.move(new Movement(10000, 'KM'));
    }
}
```

## Interface Segregation Principle - 介面隔離原則
> Clients should not be forced to depend upon interfaces that they don't use.

延續前一個範例，如果`WarShip`要多一個`Fire`的功能，且這個功能和戰車的`Fire`相似，都是要先瞄準再發射炮彈。 我該怎麼樣共同管理這類型的功能呢？

首先，先建立一個介面`IAttack`。
- IAttack.java
``` Java
interface IAttack {
    public void aim(Target target);
    public void fire();
}
```

接著，在你的`WarShip`類別後面去實作這個介面的功能。
- WarShip.java
``` Java
public class WarShip extends Ship implements IAttack {
    public void aim(Target target) {
        // Logic of aim method.
    }
    public void fire() {
        // Logic of fire method.
    }
}
```

這樣一來，假設之後我又多了一個類別`SpaceShip`也需要裝載砲彈系統。 我就可以直接去實作那個介面，不用擔心在使用的時候會發生功能名稱錯誤的情況。


## Dependency Inversion Principle - 相依反轉原則
> - High-level modules should not depend on low-level modules. Both should depend on abstractions.
> - Abstractions should not depend on details. Details should depend on abstractions.

這個原則的前提，一定要滿足前一個原則(介面隔離原則)，以確保底層的功能有按照上層定義的行為實作。
我們再繼續前一個例子，如果你要組一個戰爭艦隊，艦隊中的任何一艘船隻都可以瞄準再發射炮彈，即實作了`IAttack`。 我在操控這些船隻**同時間**發射的時候，要透過什麼樣的方法去下達發射的指令？

這邊的`IAttack[]`就是被實體倚賴的目標，它是抽象的。 
1. 只要是有攻擊能力的戰艦，都會相依在這個介面上，在`fullAttack`裡面直接被操控。
2. 每艘有攻擊能力的戰艦，都會在自己的類別中，去設定要瞄準的方法、發射的砲彈規格等等。

- World.java
``` Java
public class World {
    private void fullAttack(IAttack[] attacks, Target target) {
        for(int i = 0; i < attacks.length; i++) {
            attacks[i].aim(target);
            attacks[i].fire();
        }
    }
    public static void main(String[] args) {
        IAttack[] attacks = new IAttack[2];
        attacks[0] = new WarShip();
        attacks[1] = new SpaceShip();
        fullAttack(attacks, new Target('Mars'));
    }
}
```