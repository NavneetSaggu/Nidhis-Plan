# Aggregation vs Composition

<aside>
💡 *Aggregation*: object A contains objects B; B can live without A.

*Composition*: object A consists of objects B; A manages life cycle of B; B can’t live without A.

</aside>

| **Aggregation** | **Composition** |
| --- | --- |
| Object A contains objects B; B can live without A. | Object A consists of objects B; A manages life cycle of B; B can’t live without A. |
| A parent class is not responsible for creating or destroying the child class. | The parent class is responsible for creating or destroying the child class. |
| Aggregation is a weak Association. | Composition is a strong Association. |