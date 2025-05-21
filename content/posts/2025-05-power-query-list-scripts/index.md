---
title: Power Query Cheat Scripts I - Lists
summary: 
categories: ["Excel","Power Query", "Data Skills"]
series: ["Excel Tutorials"]
series_order: 1
tags: ["Excel","Power Query", "Data Skills"]
#externalUrl: ""
#showSummary: true
date: 2025-05-21
draft: false
---
```sql
let 
// ---------------------------------------------------------------------------
// Merging List
ListZip = List.Zip({{1..4}, {3..5},{2..7}}), // A List of lists
ListUnion = List.Union(
    {
        {1,2},{1,3},{1,4,1}
    }
    ), // Union repeats 1 in this case, why because the two 1s appeared in the same list. 

Comaprar1 = List.Union({{ "apple", "BANANA" }, {"APPLE", "banana"}}, Comparer.OrdinalIgnoreCase),
//// Output: {"apple", "banana"}
Comparer2 = List.Union( { { "apple", "banana" }, { "APPLE", "BANANA" } }, Comparer.Ordinal ), 
// Output: { "apple", "APPLE", "banana", "BANANA" }
Comparer3 = List.Union({{"Færdig", "FÆrdig" }, {"Faerdig"}}, Comparer.FromCulture("en-US", true )),
// Output: {"Færdig", "FÆrdig"}
// ---------------------------------------------------------------------------
// Recursive functions 
List_Generate1 = List.Generate(()=>0, (_)=>_< 10, (_)=>_+2),
List_Generate2 = List.Generate(() =>0, each _ < 10, each _+2), //  (_)=> can be replaced by each
List_Generate3 =  List.Generate(
    ()=> 1, 
    each _<6, 
    each _+1, 
    each Number.Factorial(_) //optional selector. the result of function can be modified
),
List_Generate4 =  List.Generate(
    () => [x = 0, y = 1], 
    each [x] < 10, 
    each [x = [y], y = [x]+[y]], 
    each [x]
),
List_Generate5 =  List.Generate(
    () => [x = 0, y = 1], 
    each [x] < 10, 
    each [x = [y], y = [x]+[y]], 
    each [y]
),
List_Generate6 =  List.Generate(
    () => [x = 0, y = 1], 
    each [x] < 10, 
    each [x = [y], y = [x]+[y]], 
    each [y]+[y]
)
,
// ---------------------------------------------------------------------------
// List.TransformMany can be used to change or transform values in a list by 
// specific logic as follows:
    List_Transform_Many = 
     List.TransformMany(
        {1,2,3},
        (x) => {2*x}, // collectionTransform as function, the second list (y) is {2,4,6}. 
        (x,y) => (Text.From(x) & "-" & Text.From(y)) //  resultTransform as function
        // the function defined in the 
        // second input is in the form of (x) =>, and its result is 
        // considered as y. In the third input of this function, a 
        // new function is defined in the form of (x, y) =>.
        // Considering these two lists, the result of the formula is {“1-2”,“2-4”,“3-6”}.
        // Fron a list -> create second list -> and based on these two list, create final list
        ),
// --------------------------------------------------------------------------- 
     List_SingleOrDefault0 =  List.SingleOrDefault({1}),
     List_SingleOrDefaultNull = List.SingleOrDefault({}),
     List_SingleOrDefault1 =  List.SingleOrDefault({1..4}), // error, default not provided
     List_SingleOrDefault2 = List.SingleOrDefault({1..4}, -1) ,
     List_SingleOrDefault3 = List.SingleOrDefault({1}, -1)  ,
     List_SingleOrDefault4 = List.SingleOrDefault({}, -1)    
    
in
    List_SingleOrDefault4
```