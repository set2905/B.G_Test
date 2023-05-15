# B.G_Test
## Задача 1:
**Изучите перк TOUGH_BODY и создайте перк, который с вероятностью
1/4 в 3 раза уменьшает урон, полученный хозяином перка от критического удара в голову.**

Создан перк CRITICAL_RESIST_HEAD
```
Perk CRITICAL_RESIST_HEAD
{
  defenseDamageFactor=-15850
  chance=0.25
  iconFrames=90
  defense="HeadDefense"

  Trigger CRITICAL_RESIST_HEAD
  {
    Event
    {
      return "PreHit"
    }
    
    Condition
    {
      return AND ( event.Critical, event.Target=="Me", event.Defense==defense, random() < chance )
    }
    
    Action
    {
      ModAttributes ( Player="Enemy", DamageFactor=defenseDamageFactor, Frames=1 )
      ModIcon ( Name="Icon", Frames=iconFrames )
    }
  }
}
```
## Задача 2:
**Изучите перк FITNESS и измените его так, чтобы он, не меняя вероятности срабатывания при ударе без оружия, с вероятностью 0.02 в 2 раза увеличивал урон по противнику любым ударом.**

Перк изменен следующим образом: добавлен триггер, который с 2% шансом при любой атаке добавляет мод на 2х урон. Например, при одновременном срабатывании двух триггеров с вероятностями 2% и 30% при рукопашной атаке - урон увеличивается в 4 раза.
```
Trigger FITNESS_ATTACK_ANY
  {
    Event
    {
      return "PreHit"
    }
    
    Condition
    {
      return AND ( event.Target=="Enemy", random() < attackAnyBoostChance )
    }
    
    Action
    {
      ModAttributes ( DamageFactor=attackDamageFactor, Frames=1, Player="Me" )
    }
  }
```
## Задача 3:
**Изучите HELM_BREAKER. Измените его так, чтобы он не мог сработать в первые
10 секунд раунда, а действие усиления в первую секунду после приобретения было
удвоено.**

При начале боя триггер HELM_BREAKER_INIT добавляет на 10 секунд мод Denied, запрещающий усиление.
Триггер усиления разделен на два: HELM_BREAKER_DOUBLEBOOST (HELM_BREAKER_DOUBLEBOOST_END) с удвоенным действием и старый - HELM_BREAKER_BOOST с измененным условием, он активируется при исчезновении мода HELM_BREAKER_DOUBLEBOOST.

## Задача 4:
**Изучите перк RAGE. Корректен ли он? Если нет, то укажите, в чем заключается ошибка и постарайтесь исправить её.**

- RAGE_CHECK_FAIL и RAGE_CHECK_SUCCESS проверяют разные возвращаемые значения random(), поэтому могут выполниться(или не выполниться) действия обоих триггеров. 
- В триггере RAGE_SUCCESS_DAMAGE используется константа defenseDamageFactor, которая не определена.
- В триггерах RAGE_SUCCESS_ICON, RAGE_CHECK_FAIL, RAGE_CHECK_SUCCESS функции ModExists, CreateEvent должны использовать именованные параметры.

Теперь, в случае успеха, на один кадр создается мод Success, а триггер RAGE_CHECK_FAIL в условии проверяет отсутствие этого мода.

## Задача 5:
**Изучите перк SHIELDING. Опишите, как он работает. Сделайте шанс срабатывания пропорциональным урону.**

Перк, при получении урона, с шансом 30% на 5 секунд уменьшает наносимый оппонентом урон в два раза. Эффект перка не может складываться и обновляться при получении урона во время его действия.

Чтобы сделать шанс срабатывания пропорциональным урону необходимо чтобы событие PostHit передавала свойство Damage - нанесенный ударом урон.

Шанс срабатывания не получится сделать пропорциональным урону имеющимися средствами. Близкое решение - изменять шанс срабатывания в зависимости от того, куда пришелся удар - в голову или в тело, был ли он критическим, был ли блок. 
Например:
```
Perk SHIELDING
{
  damageFactor=-10000
  frames=300
  baseChance=0.3
  headChance=0.5
  criticalBodyChance=0.7
  criticalHeadChance=0.9 


  Trigger SHIELDING_CRITICAL_HEAD
  {
    Event
    {
      return "PostHit"
    }
    
    Condition
    {
    return AND ( event.Target=="Me", event.Critical, event.Defense==”HeadDefense”,
    random () < criticalHeadChance, not ModExists ( "Shielding" )  )
    }
    
    Action
    {
      ModAttributes ( Name="Shielding", DamageFactor=damageFactor, Frames=frames, Player="Enemy" )
      ModIcon ( Name="Icon", Frames=frames )
    }
  }

  Trigger SHIELDING_CRITICAL_BODY
  {
    Event
    {
      return "PostHit"
    }
    
    Condition
    {
    return AND ( event.Target=="Me", event.Critical, event.Defense==”BodyDefense”,
    random () < criticalBodyChance, not ModExists ( "Shielding" )  )
    }
    
    Action
    {
      ModAttributes ( Name="Shielding", DamageFactor=damageFactor, Frames=frames, Player="Enemy" )
      ModIcon ( Name="Icon", Frames=frames )
    }
  }

  Trigger SHIELDING_HEAD
  {
    Event
    {
      return "PostHit"
    }
    
    Condition
    {
    return AND ( event.Target=="Me", not event.Critical, event.Defense==”HeadDefense”,
    random () < headChance, not ModExists ( "Shielding" )  )
    }
    
    Action
    {
      ModAttributes ( Name="Shielding", DamageFactor=damageFactor, Frames=frames, Player="Enemy" )
      ModIcon ( Name="Icon", Frames=frames )
    }
  }

  Trigger SHIELDING_OTHER
  {
    Event
    {
      return "PostHit"
    }
    
    Condition
    {
    return AND ( event.Target=="Me", not event.Critical,
    random () < baseChance, not ModExists ( "Shielding" )  )
    }
    
    Action
    {
      ModAttributes ( Name="Shielding", DamageFactor=damageFactor, Frames=frames, Player="Enemy" )
      ModIcon ( Name="Icon", Frames=frames )
    }
  }
}
```



