# Salus Reports

Salus compiles a report while executing each scanner and before finishing, this report is output in one or multiple ways.

By default, Salus always prints the report to STDOUT. This can be turned off via the `--quiet` (`-q`) flag. Additionally, Salus can be configured to send reports to both the local file system or remote HTTP endpoints. See the [configuration documentation](configuration.md) for more detail.

Salus reports contain the following information:
  - Report format version - important for report consumers to correctly determine how to parse the report.
  - Data about the codebase it scanned.
  - The configuration that Salus used during the scan.
  - The outcome of each scanner it ran.
  - Information about the repo that might be useful for further analysis (such as what dependencies it has).
  - Errors that happened during Salus execution.

## Salus Report Format

The report, structurally, is a Hash of the following form:

```
{
  "version": <String, semantic version of this report format>,
  "project_name": <String, comes from Salus configuration, typically used for identifying the codebase for this report>,
  "custom_info": <String or hash, comes from Salus configuration, used to pipe additional data from Salus runner to report consumer>,
  "configuration": <Hash, exact configuration used while Salus executed>,
  "scans": {
    <String, scanner name>: <Boolean, true for pass, false for fail>
  },
  "info": {
    <String, info type>: [
      <String, info>
    ]
  },
  "errors": {
     <String, error origin>: [
       <String, error message>
     ]
  }
}
```

While the report above is in JSON form, Salus reports can also be generated in YAML, text, SARIF and CycloneDX form.

### CycloneDX Reports
CycloneDX is a lightweight software bill of materials (SBOM) standard designed for use in application security contexts and supply chain component analysis. Salus can leverage CycloneDX to standardize dependency reporting. For more information see [CycloneDX Spec](https://cyclonedx.org/).

### Salus Scanners with CycloneDX Support
The following Salus dependency scanners are currently planned to adapt to report CycloneDX output:

- ReportGoDep
- ReportRubyGems
- ReportNodeModules 
- ReportPythonModules 
- ReportRustCrates

### Example CycloneDX Report
Below is an example of the structure of a CycloneDX report which will then be base64 encoded in the final json.

```
{
  "bomFormat": "CycloneDX",
  "specVersion": "1.3",
  "serialNumber": UUID for the BOM,
  "version": 1 (always set to 1),
  "components": [
    {
      "bom-ref": "pkg:golang/github.com/DataDog/datadog-go@v4.2.0",
      "type": "library" (Other options are application, framework, operating-system, device or file),
      "name": "github.com/DataDog/datadog-go",
      "version": "v4.2.0" (component version),
      "purl": "pkg:golang/github.com/DataDog/datadog-go@v4.2.0" 
    }
  ]
}
```

Final json output:
```
{
  "autoCreate": true,
  "projectName": "<String, project name>",
  "projectVersion": "1",
  "bom": "<base64 encoded cyclonedx report>"
}

```


### SARIF Reports
SARIF is a standardized output format for static analysis tools. Considering the wide range of security tools that Salus currently supports, with each tool producing its own unique output; SARIF provides a generalized report format for all these tools. For more information about the SARIF format visit the [SARIF website](https://docs.oasis-open.org/sarif/sarif/v2.1.0/os/sarif-v2.1.0-os.html).

SARIF reports generated by Salus follow this format: 
```
{
  "version": <String, SARIF version>,
  "$schema": <String, SARIF schema>,
  "runs": [
    {
      "tool": {
        "driver": {
          "name": <String, Scanner name>,
          "version": <String, version of scanner used to generate report>,
          "informationUri": <String, Link to the scanner's webpage>,
          "rules": [
            <reportingDescriptor(@3.49), unique objects for defining rules that were violated in the scan report>
          ]
        }
      },
      "conversion": {
        "tool": {
          "driver": {
            "name": "Salus",
            "informationUri": <String, Link to Salus project>
          }
        }
      },
      "results": [
        <resultObjects(@3.27), unique objects that define vulnerabilites highlited by a scanner after a scan>
      ],
      "invocations": [
        {
          "executionSuccessful": <Boolean, true if scanner ran successfuly, false if otherwise>
        }
      ]
    }
  ]
}
```

### Salus ID's for SARIF
| ID   | Description |
| ----------- | :---------: |
| SAL001      | This Scanner Currently does not support Sarif    |
| SAL002      | Errors logged in a scanners json report     |
| SAL003      | Errors logged by scanner during invocation |

### Salus Scanners with SARIF support 
The following salus scanners are currently adapted to report SARIF output:
- Gosec
- Yarn Audit
- NPM audit

### Example SARIF report

Report generated by scanning [iglo](https://github.com/subosito/iglo.git)
```
{
  "version": "2.1.0",
  "$schema": "https://schemastore.azurewebsites.net/schemas/json/sarif-2.1.0-rtm.5.json",
  "runs": [
    {
      "tool": {
        "driver": {
          "name": "Gosec",
          "version": "2.4.0",
          "informationUri": "https://github.com/securego/gosec",
          "rules": [
            {
              "id": "G203",
              "name": "CWE-79",
              "fullDescription": {
                "text": "this method will not auto-escape HTML. Verify data is well formed. \nSeverity: MEDIUM\nConfidence: LOW\nCWE: https://cwe.mitre.org/data/definitions/79.html"
              },
              "helpUri": "https://cwe.mitre.org/data/definitions/79.html",
              "help": {
                "text": "More info: https://cwe.mitre.org/data/definitions/79.html",
                "markdown": "[More info](https://cwe.mitre.org/data/definitions/79.html)."
              }
            },
            {
              "id": "SAL002",
              "name": "Golang Error",
              "fullDescription": {
                "text": "Golang errors generated at runtime"
              },
              "helpUri": "https://github.com/coinbase/salus/blob/master/docs/salus_reports.md",
              "help": {
                "text": "More info: https://github.com/coinbase/salus/blob/master/docs/salus_reports.md",
                "markdown": "[More info](https://github.com/coinbase/salus/blob/master/docs/salus_reports.md)."
              }
            }
          ]
        }
      },
      "conversion": {
        "tool": {
          "driver": {
            "name": "Salus",
            "informationUri": "https://github.com/coinbase/salus"
          }
        }
      },
      "results": [
        {
          "ruleId": "G203",
          "ruleIndex": 0,
          "level": "error",
          "message": {
            "text": "this method will not auto-escape HTML. Verify data is well formed. \nSeverity: MEDIUM\nConfidence: LOW\nCWE: https://cwe.mitre.org/data/definitions/79.html"
          },
          "locations": [
            {
              "physicalLocation": {
                "artifactLocation": {
                  "uri": "/home/repo/formatter.go",
                  "uriBaseId": "%SRCROOT%"
                },
                "region": {
                  "startLine": 32,
                  "startColumn": 9,
                  "snippet": {
                    "text": "31: \tb := bf.MarkdownCommon([]byte(str))\n32: \treturn template.HTML(string(b))\n33: }\n"
                  }
                }
              }
            }
          ]
        },
        {
          "ruleId": "SAL0002",
          "ruleIndex": 3,
          "level": "note",
          "message": {
            "text": "could not import github.com/subosito/iglo (invalid package name: \"\")"
          },
          "locations": [
            {
              "physicalLocation": {
                "artifactLocation": {
                  "uri": "/home/repo/examples/api-exporter/main.go",
                  "uriBaseId": "%SRCROOT%"
                },
                "region": {
                  "startLine": 8,
                  "startColumn": 2,
                  "snippet": {
                    "text": ""
                  }
                }
              }
            }
          ]
        }
      ],
      "invocations": [
        {
          "executionSuccessful": false
        }
      ]
    },
    {
      "tool": {
        "driver": {
          "name": "Semgrep",
          "version": "0.62.0",
          "informationUri": "https://github.com/coinbase/salus",
          "rules": [

          ]
        }
      },
      "conversion": {
        "tool": {
          "driver": {
            "name": "Salus",
            "informationUri": "https://github.com/coinbase/salus"
          }
        }
      },
      "results": [

      ],
      "invocations": [
        {
          "executionSuccessful": true,
          "toolExecutionNotifications": [
            {
              "descriptor": {
                "id": "SAL001"
              },
              "message": {
                "text": "SARIF reports are not available for this scanner"
              }
            }
          ]
        }
      ]
    }
  ]
}
```

## Example Reports

Reports generated by scanning the [rails/actioncable-examples@916910c](https://github.com/rails/actioncable-examples) repository.

__Text from STDOUT:__

```
#################### Salus Scan v1.0.0 for Project ####################

	Brakeman => passed

	BundleAudit => failed

STDOUT:
[
  {
    "type": "UnpatchedGem",
    "name": "loofah",
    "version": "2.0.3",
    "cve": "CVE-2018-8048",
    "url": "https://github.com/flavorjones/loofah/issues/144",
    "advisory_title": "Loofah XSS Vulnerability",
    "description": "Loofah allows non-whitelisted attributes to be present in sanitized\noutput when
 input with specially-crafted HTML fragments.\n",
    "cvss": null,
    "osvdb": null,
    "patched_versions": [
      ">= 2.2.1"
    ],
    "unaffected_versions": [
    ]
  },
  ...

	PatternSearch => passed

	RepoNotEmpty => passed

	ReportRubyGems

	overall => failed
```

__YAML Form:__

Long arrays have been shortened for brevity.

```yaml
---
:project_name: Project
:scans:
  Brakeman:
    passed: true
  BundleAudit:
    passed: false
    info:
      unpatched_gem:
      - :type: :UnpatchedGem
        :name: loofah
        :version: 2.0.3
        :cve: CVE-2018-8048
        :url: https://github.com/flavorjones/loofah/issues/144
        :advisory_title: Loofah XSS Vulnerability
        :description: |
          Loofah allows non-whitelisted attributes to be present in sanitized
          output when input with specially-crafted HTML fragments.
        :cvss:
        :osvdb:
        :patched_versions:
        - ">= 2.2.1"
        :unaffected_versions: []
      - :type: :UnpatchedGem
        :name: sprockets
        :version: 3.7.1
        :cve: CVE-2018-3760
        :url: https://groups.google.com/forum/#!topic/ruby-security-ann/2S9Pwz2i16k
        :advisory_title: Path Traversal in Sprockets
        :description: |
          Specially crafted requests can be used to access files that exist on
          the filesystem that is outside an application's root directory, when the
          Sprockets server is used in production.

          All users running an affected release should either upgrade or use one of the work arounds immediately.

          Workaround:
          In Rails applications, work around this issue, set `config.assets.compile = false` and
          `config.public_file_server.enabled = true` in an initializer and precompile the assets.

           This work around will not be possible in all hosting environments and upgrading is advised.
        :cvss:
        :osvdb:
        :patched_versions:
        - "< 3.0.0, >= 2.12.5"
        - "< 4.0.0, >= 3.7.2"
        - ">= 4.0.0.beta8"
        :unaffected_versions: []
  PatternSearch:
    passed: true
  RepoNotEmpty:
    passed: true
  ReportRubyGems:
    info:
      dependency:
      - :dependency_file: Gemfile
        :type: ruby
        :version:
      - :dependency_file: Gemfile
        :type: bundler
        :version: !ruby/object:Gem::Version
          version: 1.15.1
      - :dependency_file: Gemfile.lock
        :type: gem
        :name: actioncable
        :version: 5.1.1
        :source: rubygems repository https://rubygems.org/ or installed locally
      - :dependency_file: Gemfile.lock
        :type: gem
        :name: actionmailer
        :version: 5.1.1
        :source: rubygems repository https://rubygems.org/ or installed locally
  overall:
    passed: false
:info: {}
:errors: {}
:version: 1.0.0
:custom_info: ''
:configuration:
  :sources:
    :configured:
    - file:///salus.yaml
    :valid:
    - file:///salus.yaml
  active_scanners:
  - Brakeman
  - BundleAudit
  - NPMAudit
  - PatternSearch
  - RepoNotEmpty
  - ReportGoDep
  - ReportNodeModules
  - ReportPythonModules
  - ReportRubyGems
  enforced_scanners:
  - RepoNotEmpty
  - Brakeman
  - BundleAudit
  - NPMAudit
  - PatternSearch
  scanner_configs: {}
  reports:
  - uri: file://./repo/salus-report.txt
    format: txt
  - uri: file://./repo/salus-report.yaml
    format: yaml
    verbose: true
  - uri: file://./repo/salus-report.json
    format: json
    verbose: true
```

__JSON Form:__

Long arrays have been shortened for brevity.

```json
{
  "project_name": "Project",
  "scans": {
    "Brakeman": {
      "passed": true
    },
    "BundleAudit": {
      "passed": false,
      "info": {
        "unpatched_gem": [
          {
            "type": "UnpatchedGem",
            "name": "loofah",
            "version": "2.0.3",
            "cve": "CVE-2018-8048",
            "url": "https://github.com/flavorjones/loofah/issues/144",
            "advisory_title": "Loofah XSS Vulnerability",
            "description": "Loofah allows non-whitelisted attributes to be present in sanitized\noutput when input with specially-crafted HTML fragments.\n",
            "cvss": null,
            "osvdb": null,
            "patched_versions": [
              ">= 2.2.1"
            ],
            "unaffected_versions": [

            ]
          },
          {
            "type": "UnpatchedGem",
            "name": "sprockets",
            "version": "3.7.1",
            "cve": "CVE-2018-3760",
            "url": "https://groups.google.com/forum/#!topic/ruby-security-ann/2S9Pwz2i16k",
            "advisory_title": "Path Traversal in Sprockets",
            "description": "Specially crafted requests can be used to access files that exist on\nthe filesystem that is outside an application's root directory, when the\nSprockets server is used in production.\n\nAll users running an affected release should either upgrade or use one of the work arounds immediately.\n\nWorkaround:\nIn Rails applications, work around this issue, set `config.assets.compile = false` and\n`config.public_file_server.enabled = true` in an initializer and precompile the assets.\n\n This work around will not be possible in all hosting environments and upgrading is advised.\n",
            "cvss": null,
            "osvdb": null,
            "patched_versions": [
              "< 3.0.0, >= 2.12.5",
              "< 4.0.0, >= 3.7.2",
              ">= 4.0.0.beta8"
            ],
            "unaffected_versions": [

            ]
          }
        ]
      }
    },
    "PatternSearch": {
      "passed": true
    },
    "RepoNotEmpty": {
      "passed": true
    },
    "ReportRubyGems": {
      "info": {
        "dependency": [
          {
            "dependency_file": "Gemfile",
            "type": "ruby",
            "version": null
          },
          {
            "dependency_file": "Gemfile",
            "type": "bundler",
            "version": "1.15.1"
          },
          {
            "dependency_file": "Gemfile.lock",
            "type": "gem",
            "name": "actioncable",
            "version": "5.1.1",
            "source": "rubygems repository https://rubygems.org/ or installed locally"
          },
          {
            "dependency_file": "Gemfile.lock",
            "type": "gem",
            "name": "actionmailer",
            "version": "5.1.1",
            "source": "rubygems repository https://rubygems.org/ or installed locally"
          }
        ]
      }
    },
    "overall": {
      "passed": false
    }
  },
  "info": {
  },
  "errors": {
  },
  "version": "1.0.0",
  "custom_info": "",
  "configuration": {
    "sources": {
      "configured": [
        "file:///salus.yaml"
      ],
      "valid": [
        "file:///salus.yaml"
      ]
    },
    "active_scanners": [
      "Brakeman",
      "BundleAudit",
      "NPMAudit",
      "PatternSearch",
      "RepoNotEmpty",
      "ReportGoDep",
      "ReportNodeModules",
      "ReportPythonModules",
      "ReportRubyGems"
    ],
    "enforced_scanners": [
      "RepoNotEmpty",
      "Brakeman",
      "BundleAudit",
      "NPMAudit",
      "PatternSearch"
    ],
    "scanner_configs": {
    },
    "reports": [
      {
        "uri": "file://./repo/salus-report.txt",
        "format": "txt"
      },
      {
        "uri": "file://./repo/salus-report.yaml",
        "format": "yaml",
        "verbose": true
      },
      {
        "uri": "file://./repo/salus-report.json",
        "format": "json",
        "verbose": true
      }
    ]
  }
}
```

#### Example with Errors

Reports generated by scanning the [coinbase/traffic_jam@2b90aa5](https://github.com/coinbase/traffic_jam) ruby gem repo.

__Text from STDOUT:__

```
#################### Salus Scan v1.0.0 for Project ####################

	PatternSearch => passed

	RepoNotEmpty => passed

	overall => passed

==================== Salus Errors ====================
	Salus - [{:type=>"NoMethodError", :message=>"undefined method `versions' for nil:NilClass", :location=>"/home/lib/salus/scanners/report_ruby_gems.rb:43:in `record_dependencies_from_gemfile'"}]
```

__YAML form verbose__

```yaml
---
:project_name: Project
:scans:
  PatternSearch:
    passed: true
  RepoNotEmpty:
    passed: true
  overall:
    passed: true
:info: {}
:errors:
  Salus:
  - :type: NoMethodError
    :message: undefined method `versions' for nil:NilClass
    :location: "/home/lib/salus/scanners/report_ruby_gems.rb:43:in `record_dependencies_from_gemfile'"
:version: 1.0.0
:custom_info: ''
:configuration:
  :sources:
    :configured:
    - file:///salus.yaml
    :valid:
    - file:///salus.yaml
  active_scanners:
  - Brakeman
  - BundleAudit
  - NPMAudit
  - PatternSearch
  - RepoNotEmpty
  - ReportGoDep
  - ReportNodeModules
  - ReportPythonModules
  - ReportRubyGems
  enforced_scanners:
  - RepoNotEmpty
  - Brakeman
  - BundleAudit
  - NPMAudit
  - PatternSearch
  scanner_configs: {}
  reports:
  - uri: file://./repo/salus-report.yaml
    format: yaml
    verbose: true
  - uri: file://./repo/salus-report.json
    format: json
    verbose: true
```

__JSON form verbose__

```json
{
  "project_name": "Project",
  "scans": {
    "PatternSearch": {
      "passed": true
    },
    "RepoNotEmpty": {
      "passed": true
    },
    "overall": {
      "passed": true
    }
  },
  "info": {
  },
  "errors": {
    "Salus": [
      {
        "type": "NoMethodError",
        "message": "undefined method `versions' for nil:NilClass",
        "location": "/home/lib/salus/scanners/report_ruby_gems.rb:43:in `record_dependencies_from_gemfile'"
      }
    ]
  },
  "version": "1.0.0",
  "custom_info": "",
  "configuration": {
    "sources": {
      "configured": [
        "file:///salus.yaml"
      ],
      "valid": [
        "file:///salus.yaml"
      ]
    },
    "active_scanners": [
      "Brakeman",
      "BundleAudit",
      "NPMAudit",
      "PatternSearch",
      "RepoNotEmpty",
      "ReportGoDep",
      "ReportNodeModules",
      "ReportPythonModules",
      "ReportRubyGems"
    ],
    "enforced_scanners": [
      "RepoNotEmpty",
      "Brakeman",
      "BundleAudit",
      "NPMAudit",
      "PatternSearch"
    ],
    "scanner_configs": {
    },
    "reports": [
      {
        "uri": "file://./repo/salus-report.yaml",
        "format": "yaml",
        "verbose": true
      },
      {
        "uri": "file://./repo/salus-report.json",
        "format": "json",
        "verbose": true
      }
    ]
  }
}
```
