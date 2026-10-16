## Phase
<!-- e.g. Phase 1 — FSM + Services -->

## Branch
<!-- e.g. phase/1-fsm-services -->

## Gate condition met
- [ ] All assigned tests pass (`pytest tests/ -v`)
- [ ] App starts without errors (`docker compose up`)
- [ ] Gate condition from my instruction file is fully checked off

## Scope check
- [ ] I only modified files listed in my instruction file
- [ ] I did NOT change `app/models.py`
- [ ] I did NOT change `app/db.py`
- [ ] I did NOT change `docker-compose.yml` or `requirements.txt`
- [ ] I did NOT add new dependencies without lead approval

## Self-review
- [ ] No hardcoded passwords, secrets, or credentials
- [ ] No debug `print()` statements left in
- [ ] All money values use `Decimal`, not `float`
- [ ] All timestamps stored UTC, displayed Asia/Dhaka
- [ ] All status changes go through `fsm.transition()` or `services.*`

## Notes for lead / next agent
<!-- Anything that changed, any assumptions made, any blockers for the next phase -->
