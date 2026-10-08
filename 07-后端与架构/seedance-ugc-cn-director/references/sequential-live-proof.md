# Sequential live proof, optional contract v1

Use this only when the creator completes one garment action, lets its result settle, and then speaks while the same visible result remains readable. It supports fixed-phone German live dialogue without simultaneous complex movement and lip-sync. It does not change the clip's beat count, macro phases, minimum proof coverage, screen-text rules, reference or market evidence, receipts, paid ledger, or duplicate-submission controls.

Eligibility is limited to current schema 1.4 / `canonical_prompt_v7`, market `DE`, voiceover `de-DE`, `delivery_mode: de_live_simple`, `camera_mode: fixed_phone`, `purpose: proof`, and `beat_role: product_claim`. Hook, CTA, styling, silent, hybrid, US and historical beats cannot use the object. The beat must retain `speech_mode: on_camera_dialogue`, `mouth_visibility: visible`, and `lip_sync_required: true`.

Add this optional object directly to the canonical beat. Times are absolute video seconds, numeric and finite, with at most millisecond precision:

```json
{
  "sequential_live_proof": {
    "contract_id": "sequential_live_proof_v1",
    "action_window_seconds": [2, 2.75],
    "settle_window_seconds": [2.75, 3],
    "speech_window_seconds": [3, 5.25],
    "speech_motion_budget": {"camera": "locked", "performer": "micro", "active_hands": "zero"},
    "proof_endpoint_visible_during_speech": true,
    "speech_endpoint": "existing waist fit allowance hangs naturally after release",
    "speech_hand_anchors": {"left": "left hand resting at left thigh", "right": "right hand resting at right thigh"},
    "claim_scope": "visible_loose_allowance",
    "allowance_displacement_cm": 2,
    "fabric_extension_forbidden": true
  }
}
```

The three windows must cover the entire parent beat continuously: action start equals beat start, action end equals settling start, settling end equals speech start, and speech end equals beat end. Minimum durations are 0.5 seconds for the single action, 0.25 seconds for settling, and 2 seconds for speech. No speech or articulating mouth motion is allowed before the speech window. The speech window additionally permits at most 3.5 words per second; all existing per-line and whole-clip word budgets still apply.

The canonical beat's existing `motion_budget` and `hand_plan` describe only the action window. Existing single-action, hand-count, stable-camera, anatomy, target and framing checks still apply. The new speech budget must use the same locked/subtle camera, still/micro performer, and `active_hands: zero`. Its two `speech_hand_anchors` must exactly match the action hand plan's `end_anchor` values. Its `speech_endpoint` must exactly match both `proof_endpoint` and `visible_endpoint`. Moving back to hide the proof before speaking, maintaining a pinch while speaking, or starting a second demonstration during speech does not satisfy this contract.

Allowed actions are `pinch_release`, `pull_release`, `raise_arm`, `turn_settle`, `walk_settle`, and `pocket_use`. The bound claim must be `assertion_level: visible`, `assertion_kind: visible_feature`, and `claim_mode: visual_only`; all existing exact evidence and claim-proof bindings remain required. Material-performance or elasticity language is rejected.

For `pinch_release` and `pull_release`, `claim_scope` must be `visible_loose_allowance`, the bound claim/proof/beat must all use `product_part_id: fit`, displacement must be greater than zero and at most 2 cm, and `fabric_extension_forbidden` must be true. Demonstrate existing garment ease only; release the garment completely so the endpoint rests under gravity. Do not bind these actions to stretch, fabric softness, opacity, or other performance claims. Write only the positive performed action in `core_action`; the serializer adds the fabric-extension prohibition from the structured field.

Other allowed actions require `claim_scope: visible_structure` and omit the two allowance-only fields. They retain the ordinary target/action and framing rules; this object does not make a pocket action valid for a neckline or a turn valid for a fabric-macro claim.

Compile the complete object without manual prompt edits. The script and B-roll conditionally project it, the prompt contains exact silent-action/settling/speech windows and the preserved endpoint, and the complete canonical beat is already bound by `canonical_beats_sha256`, the director receipt, the paid fingerprint, and the exact batch-file hash. Historical no-object renderings remain unchanged. This is a planning constraint; actual generated hand motion, proof visibility and lip-sync still require media QA.
