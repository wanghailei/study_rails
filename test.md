# Test



**Use integration style controller tests over functional style controller tests.**

Rails discourages the use of functional tests in favor of integration tests (use ActionDispatch::IntegrationTest).

Integration style controller tests perform actual requests, whereas functional style controller tests merely simulate a request.

New Rails applications no longer generate functional style controller tests and they should only be used for backward compatibility.
