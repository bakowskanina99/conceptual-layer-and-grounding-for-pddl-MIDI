# Sibling-discriminability verifier

Reference implementation and test ontologies for the paper *An Automated Verifier
for Sibling Discriminability in Goal-Dependent Planning-Domain Restriction*
(MIDI 2026; citation details will be added after publication).

## What it checks

In a typed conceptual system every type has a *contract*: required and forbidden
feature-value pairs. Two sibling types (types with the same parent) are
*discriminable* if their contracts conflict on some categorical feature, that is, a
value required by one is required differently by the other, or required by one and
forbidden by the other. Such a conflict is a discriminating witness. A sibling pair
without a witness violates sibling discriminability: an object can satisfy both
contracts, and classification becomes ambiguous.

The script reads an OWL 2 DL ontology in Turtle and reports two things separately:

* `violations`: sibling pairs of internally realizable types without a witness;
* `unrealizable`: types whose own contract is contradictory, which are excluded from
  the pair check. The checked cases are (1) the same pair both required and
  forbidden, (2) two different required values of one feature, and (3) a contract
  that forbids every value of a feature (this case needs the optional domains file).

## Files

| File | Content |
|---|---|
| `pipeline/verify_sibling_discriminability.py` | the verifier |
| `pipeline/ontology/well_formed.ttl` | a well-formed 11-type hierarchy |
| `pipeline/ontology/weakened.ttl` | the same hierarchy with discriminating features removed from most sibling groups |
| `results/variant-D-ablation-full-results.md` | results of the ablation study the weakened ontology comes from |
| `worked-example/ontology/weakened_variant_D.md` | how the weakened ontology was built |
| `worked-example/ontology/well_formed_and_encoding.md` | the well-formed ontology and its OWL encoding |

## Requirements and usage

Python 3 and `rdflib` (`pip install rdflib`). Tested with Python 3.14.6
and rdflib 7.6.0.

```
python pipeline/verify_sibling_discriminability.py pipeline/ontology/well_formed.ttl
python pipeline/verify_sibling_discriminability.py pipeline/ontology/weakened.ttl
```

Optional, template (replace the placeholders with your own files):

```
python pipeline/verify_sibling_discriminability.py --domains DOMAINS.json FILE.ttl
```

`DOMAINS.json` is optional. It maps a feature IRI to the full list of its values, for
example `{"http://example.org/store#size": ["small", "large"]}`, and enables case (3).

## Expected results

`well_formed.ttl`: 13 sibling pairs checked, 0 violations.

```
=== pipeline/ontology/well_formed.ttl ===
Pairs checked: 13
Violations (0):
```

`weakened.ttl`: 13 sibling pairs checked, 13 violations (every checked pair). The order of the
violation lines can differ from run to run.

```
=== pipeline/ontology/weakened.ttl ===
Pairs checked: 13
Violations (13):
  DairyProduct / FreshProduce
  DairyProduct / MeatProduct
  DairyProduct / PackagedFood
  FreshProduce / MeatProduct
  FreshProduce / PackagedFood
  MeatProduct / PackagedFood
  ClothingItem / ElectronicsItem
  ClothingItem / GroceryItem
  ClothingItem / HouseholdItem
  ElectronicsItem / GroceryItem
  ElectronicsItem / HouseholdItem
  GroceryItem / HouseholdItem
  Fruit / Vegetable
```

Neither shipped ontology uses forbidden values, so they exercise only the
required-value conflicts. The forbidden-value cases and the contract precheck are
implemented but are not covered by these two files.

## Scope

* Categorical features only; boolean features count as categorical. Numeric
  restrictions are out of scope.
* Contracts are read from the asserted axioms: `owl:hasValue` restrictions inside the
  class's `owl:equivalentClass` definition for required values, and
  `owl:hasValue` wrapped in `owl:complementOf` for forbidden values. No reasoner is
  used. Inherited restrictions, imports and inferred axioms are not materialized.
* The hierarchy is assumed to be tree-shaped. A class with several parents is not
  detected: the last parent read is used.

## Provenance of the two ontologies

Both ontologies come from an earlier ablation study of how much the discriminability
assumption matters in practice. The weakened ontology removes the discriminating
features from most sibling groups. The study's results are in
`results/variant-D-ablation-full-results.md`. The raw runs of that study are not part
of this repository.

## License

Creative Commons Attribution 4.0 International, see `LICENSE`.
