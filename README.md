# Youth Policy Schema (`yps:`)

A descriptive metadata schema for structuring youth policy benefit information.

**Live site:** `https://d0d002.github.io/metadata_YouthPolicySchema/`
**Author:** DoAh Kim (김도아) · 2021310281 · Principles of Metadata, Final Project


---

## a) Opening Narrative

### History of Metadata in This Domain

Youth policy benefits in Korea — housing subsidies, scholarships, job-training grants, mental
health support, and similar programs — are published by a wide range of national ministries,
local governments, and public agencies. There is no shared metadata standard across these
publishers: the same kind of information (eligibility age, application period, administering
agency) appears in inconsistent formats, ranging from PDF flyers to plain web pages to structured
portal listings. As a result, the information that exists is rarely re-usable across systems, and
young people seeking benefits must manually cross-reference many different sources to figure out
what actually applies to them.

This schema, the Youth Policy Schema (`yps:`), was designed to give that scattered information a
consistent descriptive structure — the same role that schemas like Dublin Core or MODS play for
documents and library objects, applied instead to a domain (government policy benefit
announcements) that has not had a comparable metadata tradition.

### Intended Users

The schema has two intended audiences. The primary audience is **young adults (roughly ages
19–34)** trying to find benefits relevant to their own situation — residence, income, employment
or student status, and household type. The secondary audience is anyone building a **youth policy
exploration service** on top of this metadata: a developer or administrator who needs
consistently structured records to power search, filtering, and a content-management workflow.

As discussed in a project consultation with the course instructor, the schema is intentionally
scoped to describing *individual policy records*, not the full service. A real service would also
need separate data structures for user accounts, recommendation logic, and administrator review —
those are out of scope for this schema, which focuses on making each policy benefit findable,
comparable, and verifiable.

### Development Notes

The schema went through four drafts before reaching its current form:

- **Draft 1** identified the domain and a working list of descriptive elements (policy name,
  target group, age, residence, income/household conditions, support content, application period
  and method, administering agency, verification date).
- **Draft 2** surveyed existing standards and vocabularies that could inform the design (see
  Section b), and defined ten exemplar policy types the schema needed to cover.
- **Draft 3** formalized the element list into an overview table, a full data dictionary, and
  controlled-vocabulary appendices — still as a paper design, not yet machine-validated.
- **Draft 4** translated Draft 3 into a working XSD. Comparing schemas with classmates in class
  led to five concrete changes:
  1. adding the missing root element `policyRecord`
  2. splitting `applicationPeriod` from one free-text field into typed `startDate` / `endDate` /
     `applicationCycle` sub-elements
  3. restricting `geographicEligibility` from an open string to a controlled
     `National` / `RegionSpecific` vocabulary
  4. changing `officialSource` from `xs:string` to `xs:anyURI`
  5. normalizing the `applicationMethod` value `"Offline (InPerson)"` to `OfflineInPerson` for
     consistent camelCase style
- **Post-revision (instructor feedback, 06/11)** — a further design question surfaced once the
  schema was tested against realistic data: collapsing `geographicEligibility` to just
  `National`/`RegionSpecific` meant the schema could filter by scope but could not record which
  actual region (e.g. 서울특별시, 부산광역시) a region-specific policy applied to. Per the
  instructor's guidance, region names in Korean government documents are not standardized enough
  to enumerate, so a sibling free-text element, `eligibleRegionName`, was added to hold the literal
  region name while `geographicEligibility` remains the controlled filtering value. The same
  consultation also confirmed adding an optional `level` attribute (`National`/`Local`) to
  `administeringAgency`, modeled with `xs:complexType`/`xs:simpleContent` so the element keeps
  free-text content while gaining a structured qualifier.

### Theoretical Background

Two design principles run through the schema. First, a consistent split between
**controlled-vocabulary fields** used for filtering (`supportType`, `targetGroup`,
`geographicEligibility`, `applicationMethod`, `householdCondition`,
`employmentOrStudentStatus`) and **free-text fields** used where real-world data is not yet
standardized enough to enumerate (`eligibleRegionName`, `supportDetail`, `incomeCondition`, agency
names). This mirrors a recurring lesson from the project: forcing inconsistent real-world text
into an enumeration too early makes a schema brittle, while leaving everything as free text makes
it unsearchable — the schema tries to draw that line element-by-element rather than globally.

Second, a distinction between **core** and **optional** elements, reflecting that not all
eligibility criteria apply to every policy (e.g. income conditions matter for need-based
scholarships but not for a universal counseling program). Provenance and currency are treated as
first-class concerns — `officialSource` and `lastVerifiedDate` are mandatory — because youth
policy details change often enough that an unverified record is of limited use. This structural
separation of identity, eligibility, benefit content, application logistics, and
administrative/provenance information was directly influenced by how museum and library schemas
(CDWA Lite, MODS) divide description from administrative metadata, even though this schema's
subject matter (policy programs rather than physical or bibliographic objects) is very different.

---

## b) Relevant Metadata Standards

Draft 2 surveyed existing schemas and vocabularies relevant to describing youth policy benefits.
These standards were not imported directly into the final XSD — most are RDF/OWL vocabularies not
designed for direct XSD import — but they shaped the schema's structure and remain useful as
conceptual models, which is why the final schema defines its own `yps:` namespace
(`http://example.org/yps/`) rather than reusing external element names.

| Schema / Vocabulary | Prefix | Namespace | Why it was considered |
|---|---|---|---|
| Dublin Core Terms | `dcterms` | `http://purl.org/dc/terms/` | Provides a baseline for general descriptive elements such as title, description, source, and date — the core identity information every policy record needs. |
| SKOS (Simple Knowledge Organization System) | `skos` | `http://www.w3.org/2004/02/skos/core#` | Informed how to structure controlled vocabularies such as support type, employment status, household type, and regional classification as constrained, reusable value sets. |
| Schema.org | `schema` | `https://schema.org/` | Offers flexible patterns for describing public-facing information like services, organizations, audiences, and web resources — relevant since policy benefits are essentially public services. |
| W3C Organization Ontology | `org` | `http://www.w3.org/ns/org#` | Each policy is tied to a government agency, local government, or institution, so this ontology's modeling of organizational structure was a useful reference for the `administeringAgency` element. |
| PROV-O (Provenance Ontology) | `prov` | `http://www.w3.org/ns/prov#` | Policy information changes frequently, so tracking where information came from and when it was last checked is essential — the basis for `officialSource` and `lastVerifiedDate`. |
| DCAT (Data Catalog Vocabulary) | `dcat` | `http://www.w3.org/ns/dcat#` | Policy information is often published through public data portals, so DCAT's model for connecting records to datasets and sources informed how records reference their official source. |

Structural inspiration was also drawn from museum/library description standards such as **CDWA
Lite**, **VRA Core**, and **MODS**, which clearly separate identity, classification, descriptive,
source, and administrative information — a division reflected in how this schema separates
eligibility criteria from application logistics and provenance fields. These were not imported as
namespaces (they describe artworks, visual resources, and bibliographic objects, not policy
programs), but their pattern of dividing a record into clear functional zones directly shaped how
`policyRecord` is organized: identity → eligibility → benefit content → application → provenance.

---

## c) Vocabulary Specification

### Element Overview Table (from Draft 3)

| Element name | Status | Brief definition |
|---|---|---|
| yps:policyTitle | Core | The official or commonly used name of the policy benefit. |
| yps:policySummary | Core | A short description of the policy and what it provides. |
| yps:supportType | Core | The general category of support offered by the policy. |
| yps:targetGroup | Core | The main intended beneficiaries of the policy. |
| yps:ageRequirement | Core | The age-based eligibility condition for the policy. |
| yps:geographicEligibility | Core | The residence- or location-based eligibility condition (controlled value: National / RegionSpecific). |
| yps:eligibleRegionName | Optional | The actual region name as written in the source document (e.g. 서울특별시). Added after instructor feedback to pair with geographicEligibility. |
| yps:supportDetail | Core | The specific form, amount, duration, or service content of the benefit. |
| yps:applicationPeriod | Core | The application window or timing information, structured as startDate / endDate / applicationCycle. |
| yps:applicationMethod | Core | The way a user can apply for the policy benefit. |
| yps:administeringAgency | Core | The organization responsible for managing or delivering the policy, with an optional level attribute (National/Local). |
| yps:officialSource | Core | The official source URL where the policy information is published. |
| yps:lastVerifiedDate | Core | The date on which the policy information was last checked or updated. |
| yps:incomeCondition | Optional | The income-based eligibility condition, if relevant. |
| yps:householdCondition | Optional | The household or family-related eligibility condition, if relevant. |
| yps:employmentOrStudentStatus | Optional | The applicant's employment or student status, if relevant to eligibility. |

### Controlled Vocabulary Appendices

| Appendix | Element | Allowed values |
|---|---|---|
| A1 | yps:supportType | Housing, CashAllowance, Scholarship, Employment, Training, Entrepreneurship, SavingsAssetBuilding, Transportation, HealthCounseling, CultureLeisure, LoanInterestSupport, Mixed |
| A2 | yps:targetGroup | GeneralYouth, CollegeStudents, JobSeekers, YoungWorkers, YoungEntrepreneurs, OnePersonHouseholds, LowIncomeYouth, RegionalResidents, NewlyEmployedYouth |
| A3 | yps:applicationMethod | Online, OfflineInPerson, Mail, Mixed |
| A4 | yps:employmentOrStudentStatus | Student, JobSeeker, Employed, SelfEmployed, Entrepreneur, Unspecified |
| A5 | yps:householdCondition | OnePersonHousehold, MultiPersonHousehold, DependentHousehold, LowIncomeHousehold, Unspecified |
| A6 (Draft 4) | yps:geographicEligibility | National, RegionSpecific |

### Full Data Dictionary

**1. yps:policyTitle**
| Field | Content |
|---|---|
| Label | Policy Title |
| URI | http://example.org/yps/policyTitle |
| Full definition | The official or commonly used name that identifies a youth policy benefit record; the primary human-readable title of the policy. |
| Data values | String |
| Controlled vocabulary | None |
| Cardinality | Mandatory: Yes / Repeatable: No |
| Example | "Youth Monthly Rent Support Program" |

**2. yps:policySummary**
| Field | Content |
|---|---|
| Label | Policy Summary |
| URI | http://example.org/yps/policySummary |
| Full definition | A concise narrative summary explaining the policy's purpose and the kind of benefit or service it provides. |
| Data values | String |
| Controlled vocabulary | None |
| Cardinality | Mandatory: Yes / Repeatable: No |
| Example | "A housing support program for young adults who live independently and pay monthly rent." |

**3. yps:supportType**
| Field | Content |
|---|---|
| Label | Support Type |
| URI | http://example.org/yps/supportType |
| Full definition | The broad category or categories of support provided by the policy benefit, used to compare policies by benefit type. |
| Data values | Controlled term |
| Controlled vocabulary | See Appendix A1 |
| Cardinality | Mandatory: Yes / Repeatable: Yes |
| Example | Housing; CashAllowance |

**4. yps:targetGroup**
| Field | Content |
|---|---|
| Label | Target Group |
| URI | http://example.org/yps/targetGroup |
| Full definition | The intended beneficiary group or user category that the policy is designed to support. |
| Data values | Controlled term |
| Controlled vocabulary | See Appendix A2 |
| Cardinality | Mandatory: Yes / Repeatable: Yes |
| Example | JobSeekers; CollegeStudents |

**5. yps:ageRequirement**
| Field | Content |
|---|---|
| Label | Age Requirement |
| URI | http://example.org/yps/ageRequirement |
| Full definition | The age-based eligibility rule that determines whether an applicant's age falls within the required range. |
| Data values | String (numeric range or "NoAgeLimit") |
| Controlled vocabulary | Recommended format: "NN-NN" or "NoAgeLimit" |
| Cardinality | Mandatory: Yes / Repeatable: No |
| Example | "19-34" |

**6. yps:geographicEligibility**
| Field | Content |
|---|---|
| Label | Geographic Eligibility |
| URI | http://example.org/yps/geographicEligibility |
| Full definition | The residence/location-based eligibility condition. Restricted in Draft 4 to a controlled vocabulary to prevent inconsistent free-text entries; pairs with eligibleRegionName for the literal region name. |
| Data values | Controlled term |
| Controlled vocabulary | National, RegionSpecific |
| Cardinality | Mandatory: Yes / Repeatable: Yes |
| Example | National |

**6b. yps:eligibleRegionName** *(added post-Draft 4, instructor feedback)*
| Field | Content |
|---|---|
| Label | Eligible Region Name |
| URI | http://example.org/yps/eligibleRegionName |
| Full definition | The actual region name as written in the source policy document (e.g. 서울특별시, 부산광역시, 전국). Kept as free text because region naming is not standardized across agencies. |
| Data values | String |
| Controlled vocabulary | None (intentional — see Development Notes) |
| Cardinality | Mandatory: No / Repeatable: Yes |
| Example | 서울특별시 / 전국 / 경기도 |

**7. yps:supportDetail**
| Field | Content |
|---|---|
| Label | Support Detail |
| URI | http://example.org/yps/supportDetail |
| Full definition | The specific form, amount, duration, or service content of the benefit. |
| Data values | String |
| Controlled vocabulary | None |
| Cardinality | Mandatory: Yes / Repeatable: No |
| Example | "Monthly rent subsidy up to KRW 200,000 for up to 12 months." |

**8. yps:applicationPeriod**
| Field | Content |
|---|---|
| Label | Application Period |
| URI | http://example.org/yps/applicationPeriod |
| Full definition | The time range or cycle during which users may apply. Structured (Draft 4) into three optional sub-elements so each piece of timing information is independently typed. |
| Data values | Complex type: startDate (xs:date), endDate (xs:date), applicationCycle (xs:string) |
| Controlled vocabulary | applicationCycle commonly: Rolling, AnnualCycle, Quarterly |
| Cardinality | Parent mandatory: Yes / Repeatable: No; each sub-element optional |
| Example | startDate=2026-03-01, endDate=2026-11-30 — or — applicationCycle=Rolling |

**9. yps:applicationMethod**
| Field | Content |
|---|---|
| Label | Application Method |
| URI | http://example.org/yps/applicationMethod |
| Full definition | The mode(s) by which an applicant can submit an application. |
| Data values | Controlled term |
| Controlled vocabulary | See Appendix A3 |
| Cardinality | Mandatory: Yes / Repeatable: Yes |
| Example | Online; OfflineInPerson |

**10. yps:administeringAgency**
| Field | Content |
|---|---|
| Label | Administering Agency |
| URI | http://example.org/yps/administeringAgency |
| Full definition | The government body, local authority, or institution responsible for managing or delivering the policy benefit. Modeled with xs:simpleContent so it carries both free-text content and an optional level attribute. |
| Data values | String content + optional "level" attribute |
| Controlled vocabulary | level attribute: National, Local |
| Cardinality | Mandatory: Yes / Repeatable: Yes |
| Example | `<administeringAgency level="Local">Seoul Metropolitan Government</administeringAgency>` |

**11. yps:officialSource**
| Field | Content |
|---|---|
| Label | Official Source |
| URI | http://example.org/yps/officialSource |
| Full definition | The official webpage, document, or data portal where the policy information can be verified. Typed as xs:anyURI (Draft 4 change from xs:string) for explicit URI validation. |
| Data values | anyURI |
| Controlled vocabulary | None |
| Cardinality | Mandatory: Yes / Repeatable: Yes |
| Example | https://www.example.go.kr/youth-rent-support |

**12. yps:lastVerifiedDate**
| Field | Content |
|---|---|
| Label | Last Verified Date |
| URI | http://example.org/yps/lastVerifiedDate |
| Full definition | The date the policy record was last checked or updated for accuracy and currency. |
| Data values | Date (ISO 8601, YYYY-MM-DD) |
| Controlled vocabulary | None |
| Cardinality | Mandatory: Yes / Repeatable: No |
| Example | 2026-04-02 |

**13. yps:incomeCondition**
| Field | Content |
|---|---|
| Label | Income Condition |
| URI | http://example.org/yps/incomeCondition |
| Full definition | The income-based eligibility requirement, if applicable. |
| Data values | String |
| Controlled vocabulary | None (normalized expressions recommended) |
| Cardinality | Mandatory: No / Repeatable: No |
| Example | "Household income below 150% of median income." |

**14. yps:householdCondition**
| Field | Content |
|---|---|
| Label | Household Condition |
| URI | http://example.org/yps/householdCondition |
| Full definition | The household composition or family-status requirement relevant to eligibility. |
| Data values | Controlled term |
| Controlled vocabulary | See Appendix A5 |
| Cardinality | Mandatory: No / Repeatable: No |
| Example | OnePersonHousehold |

**15. yps:employmentOrStudentStatus**
| Field | Content |
|---|---|
| Label | Employment or Student Status |
| URI | http://example.org/yps/employmentOrStudentStatus |
| Full definition | The work or education status required for eligibility, if applicable. |
| Data values | Controlled term |
| Controlled vocabulary | See Appendix A4 |
| Cardinality | Mandatory: No / Repeatable: Yes |
| Example | JobSeeker; Student |

---

## Files in This Repository

- `index.html`, `style.css` — the published documentation website (sections a–e)
- `yps_schema.xsd` — the XSD with `xs:documentation` annotations and design-intent comments
- `record1_rent_support.xml`, `record2_mental_health.xml`, `record3_scholarship.xml` — sample
  records, validated against `yps_schema.xsd`
- `DEPLOY.md` — step-by-step GitHub Pages setup
