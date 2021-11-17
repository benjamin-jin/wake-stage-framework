# Wake Stagify

## Wake
[`WAKE`](https://github.com/sifive/wake) is a build orchestration tool and language developed by [SiFive](https://sifive.com) ([GitHub](https://github.com/sifive)).

## FIRRTL / Chisel3 - Like Process
[FIRRTL](https://github.com/chipsalliance/firrtl) is an open source framework to build Hardware RTL. In this project, the concept of `stage` and `phase` is used.

`Stage` has list of phases where each phase performs specific job. Each `phase` needs list of `annotations` as arguments, and returns (un)modified list of annotations.

This concept has following benefits.
- Insert a new flow into existing phase at ease.

Since all phases only require list of annotations, you do not need to consider the input types and output types of each process. Just filter your input annotations to find which ones are required, and generate new annotations at the end.

- Use previous process outputs at ease.

Stash previos phases' outputs and pop whenever needed.

## Implementation
### Annotation
`WAKE` does not support `type bound`, hence it is impossible to make list of different type elements (such as list of string and number). Hence, a new data type is required to make list of annotation.
```
tuple Annotation
```

In order to differentiate each annotation, the string type of `name` field is required.
```
tuple Annotation
	Name : String
```
Moreover, different set of value type should be required, whereas their initial values should be unconsidered. This can be resolved by wrapping each value with `Option`.

```
tuple Annotation =
  Name : String
  export String  : Option String
  export Path    : Option Path
  export Job     : Option Job
  export Integer : Option Integer
  export Double  : Option Double
  export Boolean : Option Boolean
```


### Phase
Two fields are needed for `phase instance`.
- (optional) name of phase
- (required) transform function
```
tuple Phase =
  export Name          : String
  export Transform     : List Annotation => List Annotation
```

### Stage
```
tuple Stage =
  export Name : String
  export Phases : List Phase
  export Wrapper : List Phase
```

### Execution of Stage
```
export def execute_stage stage start_annotations =
 	stages.getStagePhases | foldl (\annos\phase (phase.getPhaseTransfrom annos)) start_annotations
```

