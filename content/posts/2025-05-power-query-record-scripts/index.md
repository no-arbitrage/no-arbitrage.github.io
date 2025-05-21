---
title: Power Query Cheat Scripts II - Records
summary: 
categories: ["Excel","Power Query", "Data Skills"]
series: ["Excel Tutorials"]
series_order: 2
tags: ["Excel","Power Query", "Data Skills"]
#externalUrl: ""
#showSummary: true
date: 2025-05-21
draft: false
---
```sql
let
// ---------------------------------------------------------------------------
// Define a record
MixTyoe = [A=5,B=A+3,C=A+B], 
WithOperation =  [A=5,B=A+3,C=A+B],
MixTypeWithOpeartion = [A=5,B={1..3},C=[A=1,B=3],D={2}],
Merge_records = [A=5,B=3] & [A=2,C=4] ,

// ---------------------------------------------------------------------------
// Record properties
Record_FieldCount =  Record.FieldCount([A=5,B={1..3},C=[A=1,B=3],D={2}]),
Record_FieldName= Record.FieldNames([Name="Omid",Family="Motamedi"]),
Record_FieldValue = Record.FieldValues([A=5,B=2]),
Record_ToList =  Record.ToList([A=5,B={1..3},C=10]),
ExtractFieldValue = Record.Field([A=5,B=2],"A"),
ExtractListOfFields0 = Record.SelectFields([A=5,B=2,C=10,D=12],{"A","C","N"}),
ExtractListOfFields1 = Record.SelectFields([A=5,B=2,C=10,D=12],{"A","C","N"},1),
ExtractListOfFields2 = Record.SelectFields([A=5,B=2,C=10,D=12],{"A","C","N"},2),
WithDefaultOption =  Record.FieldOrDefault([A=5,B=2],"C"),
WithDefaultOption2 = Record.FieldOrDefault([A=5,B=2],"C",-1),
CheckFields = Record.HasFields([A=5,B=2,C={1..4}],{"A","C"}),

// ---------------------------------------------------------------------------
// Records manipulation
RemoveFields =  Record.RemoveFields([A=5,B=2,C=10,D=2*C,E=1],{"A","B","C","X"}, 1),
Rename1 = Record.RenameFields([A=5,B=2,C=10],{"A","X"}), //[X=5,B=2,C=10]
Rename2 = Record.RenameFields([A=5,B=2,C=10],{{"A","X"}, {"B","W"}}) ,// [X=5,W=2,C=10]
Rename3 = Record.RenameFields([A=5,B=2,C=10],{"U","X"}),//error
Rename4 = Record.RenameFields([A=5,B=2,C=10],{"U","X"},1), //[a=5,B=2,C=10]
Reorder1 = Record.ReorderFields([A=5,B=2,C=10],{"C","B","A"}), // [C=10,B=2,a=5]
Reorder2 = Record.ReorderFields([A=5,B=2,C=10],{"X","B", "A"}), //Error
Reorder3 = Record.ReorderFields([A=5,B=2,C=10],{"X","B", "A"},1), // [B=2,C=10,a=5]
// if the value 0 is entered in this input (or if this input is not 
// entered), the result is an error, and if the numbers 1 
// or 2 are entered, this inconsistency is ignored.
TransformFields1 =  Record.TransformFields([A="4", B=4, C="4"], {"C", Number.FromText}) ,
//Notice the quotatiin around the fieldname

// ---------------------------------------------------------------------------
// Record converstion
FromList1 = Record.FromList({1,2,3},{"A","B","C"}), // [a=1,B=2,C=3]
FromList2 = Record.FromList({1,"XWZ",3},{"A","B","C"}), // [a=1,B="XWZ",C=3]
FromList3 = Record.FromList({1,2,3}, type [A=number,B=number,C=number]), //  [a=1,B=2,C=3]
ToTable = Record.ToTable( FromList2),
FromTable = Record.FromTable(ToTable),
// ---------------------------------------------------------------------------
Final = 0
in
    Final
```