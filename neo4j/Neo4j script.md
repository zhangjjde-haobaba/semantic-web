Neo4j script

1.Load data and create node

```cypher
LOAD CSV WITH HEADERS FROM 'file:///roles.csv' AS row
CREATE (r:Role {
    project_name: row.`Project Name`,
    project_id: toInteger(row.`Project ID`),
    role_name: row.`Role Name`,
    role_description: row.`Role Description`,
    hourly_rate: toFloat(row.`Hourly Rate`),
    required_skills: split(row.`Required Skills`, ', '),
    preferred_skills: split(row.`Preferred Skills`, ', '),
    tools: split(row.`Tools`, ', '),
    availability_date: date(row.`Availability Date`),
    min_available_hours_per_week: toInteger(row.`Minimum Available Hours per Week`),
    location: row.Location,
    industry: row.industry,
    years_of_experience: toInteger(row.`Years of Experience`),
    language: row.Language
});
```

```cypher
LOAD CSV WITH HEADERS FROM 'file:///applicants.csv' AS line
MERGE (a:Applicant {UID: line.UID})
ON CREATE SET 
  a.first_name = line.`First Name`, 
  a.last_name = line.`Last Name`,
  a.location = line.Location, 
  a.availability = date(replace(replace(line.Availability, '/', '-'), '-0', '-')), 
  a.role = line.Role, 
  a.experience = toInteger(line.`Years of Professional Experience`), 
  a.skills = split(line.Skills, ', '), 
  a.social_links = line.`Social Links`, 
  a.biography = line.`Short Biography`, 
  a.personality_questions = line.`3 x Personality Questions`, 
  a.tools = split(line.Tools, ', '), 
  a.languages = split(line.Languages, ', '), 
  a.work_experience = line.`Work Experience (Title, Company, Years, Industry)`, 
  a.projects = line.Projects, 
  a.hourly_rate = toFloat(line.`Hourly Rate($)`), 
  a.working_hours = toInteger(line.`Working Hours`);
```

**Note: These two files need to be placed under the import folder**

<img src="/Users/jiangyiwei/Library/Application Support/typora-user-images/image-20230924170402687.png" alt="image-20230924170402687" style="zoom:50%;" />

2.create Tools and Skills nodes to connect these two files

```cypher
// create Skill node
MATCH (r:Role)
UNWIND r.required_skills AS skill
MERGE (s:Skill {name: skill});
MATCH (r:Role)
UNWIND r.preferred_skills AS skill
MERGE (s:Skill {name: skill});
MATCH (a:Applicant)
UNWIND a.skills AS skill
MERGE (s:Skill {name: skill});

// create Tool node
MATCH (r:Role)
UNWIND r.tools AS tool
MERGE (t:Tool {name: tool});
MATCH (a:Applicant)
UNWIND a.tools AS tool
MERGE (t:Tool {name: tool});

```

3.build relationship

```cypher

MATCH (r:Role)
UNWIND r.required_skills AS skill
MATCH (s:Skill {name: skill})
MERGE (r)-[:REQUIRES_SKILL]->(s);
MATCH (r:Role)
UNWIND r.preferred_skills AS skill
MATCH (s:Skill {name: skill})
MERGE (r)-[:PREFERS_SKILL]->(s);
MATCH (r:Role)
UNWIND r.tools AS tool
MATCH (t:Tool {name: tool})
MERGE (r)-[:REQUIRES_TOOL]->(t);


MATCH (a:Applicant)
UNWIND a.skills AS skill
MATCH (s:Skill {name: skill})
MERGE (a)-[:HAS_SKILL]->(s);
MATCH (a:Applicant)
UNWIND a.tools AS tool
MATCH (t:Tool {name: tool})
MERGE (a)-[:KNOWS_TOOL]->(t);

```

