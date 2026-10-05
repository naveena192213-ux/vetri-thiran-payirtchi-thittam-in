"""
app.py
LegalEase: AI-Powered Legal Document Generator
Streamlit Web Application
"""

import os
import streamlit as st
from datetime import datetime
from dotenv import load_dotenv

from legal_templates import TEMPLATES, render_template
from ai_engine import (
    generate_ai_contract,
    refine_document_with_ai,
    explain_in_plain_english,
    audit_contract_risks
)
from pdf_generator import generate_legal_pdf

# Load environment variables if available
load_dotenv()

# Streamlit Page Config
st.set_page_config(
    page_title="LegalEase | AI Legal Document Generator",
    page_icon="⚖️",
    layout="wide",
    initial_sidebar_state="expanded"
)

# Custom Styling
st.markdown("""
<style>
    .main-title {
        font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        color: #1E3A8A;
        font-weight: 800;
        font-size: 2.3rem;
        margin-bottom: 0.2rem;
    }
    .sub-title {
        color: #4B5563;
        font-size: 1.1rem;
        margin-bottom: 1.5rem;
    }
    .badge {
        display: inline-block;
        padding: 0.25rem 0.6rem;
        font-size: 0.8rem;
        font-weight: 600;
        border-radius: 9999px;
        background-color: #E0E7FF;
        color: #3730A3;
        margin-right: 0.5rem;
    }
    .stDownloadButton button {
        background-color: #1E3A8A !important;
        color: white !important;
        font-weight: 600 !important;
        border-radius: 6px !important;
    }
</style>
""", unsafe_allow_html=True)

# Session State Initialization
if "current_document" not in st.session_state:
    st.session_state.current_document = ""
if "current_doc_title" not in st.session_state:
    st.session_state.current_doc_title = "Legal Agreement"
if "analysis_result" not in st.session_state:
    st.session_state.analysis_result = ""

# Sidebar Configuration
with st.sidebar:
    st.image("https://images.unsplash.com/photo-1589829545856-d10d557cf95f?w=600&auto=format&fit=crop&q=80", use_container_width=True)
    st.markdown("### ⚖️ LegalEase Settings")
    
    # API Key Handling
    env_key = os.getenv("GEMINI_API_KEY", "")
    user_api_key = st.text_input(
        "Google Gemini API Key",
        value=env_key,
        type="password",
        help="Enter your Gemini API key from Google AI Studio. If left blank, LegalEase will use Smart Template Mode."
    )
    active_api_key = user_api_key.strip() if user_api_key.strip() else None

    if active_api_key:
        st.success("🟢 AI Engine: Gemini Connected")
    else:
        st.info("🟡 AI Engine: Smart Template Mode (No API key required)")

    st.markdown("---")
    navigation_mode = st.radio(
        "Navigation",
        [
            "📝 Standard Templates",
            "🤖 AI Custom Drafter",
            "🔍 Contract Risk Auditor",
            "📖 Plain English Explainer"
        ]
    )

    st.markdown("---")
    st.markdown("#### ℹ️ Disclaimer")
    st.caption(
        "LegalEase is an AI assistant intended solely for educational, administrative, and initial drafting purposes. "
        "It does not constitute formal legal counsel. Always consult a qualified attorney for legal validation."
    )

# Header Section
col_head1, col_head2 = st.columns([3, 1])
with col_head1:
    st.markdown('<div class="main-title">⚖️ LegalEase</div>', unsafe_allow_html=True)
    st.markdown('<div class="sub-title">AI-Powered Legal Document Generator & Compliance Auditor</div>', unsafe_allow_html=True)
with col_head2:
    st.markdown("""
        <div style="text-align: right; margin-top: 15px;">
            <span class="badge">Python 3</span>
            <span class="badge">Streamlit</span>
            <span class="badge">Gemini AI</span>
        </div>
    """, unsafe_allow_html=True)

st.divider()

# ==========================================
# MODE 1: STANDARD LEGAL TEMPLATES
# ==========================================
if navigation_mode == "📝 Standard Templates":
    st.subheader("📑 Standard Legal Contract Templates")
    st.write("Select a pre-vetted legal agreement framework, customize the fields, and generate your contract.")

    selected_template_name = st.selectbox("Choose Template Type:", list(TEMPLATES.keys()))
    template_info = TEMPLATES[selected_template_name]
    st.info(f"**Overview:** {template_info['description']}")

    with st.form("template_form"):
        st.markdown("#### 🖊️ Contract Parameters")
        form_values = {}
        fields = template_info["fields"]

        # Render form fields in 2 columns
        col1, col2 = st.columns(2)
        for i, field in enumerate(fields):
            curr_col = col1 if i % 2 == 0 else col2
            with curr_col:
                if field.get("type") == "select":
                    form_values[field["id"]] = st.selectbox(
                        field["label"],
                        options=field["options"],
                        index=0
                    )
                else:
                    form_values[field["id"]] = st.text_input(
                        field["label"],
                        value=field.get("default", "")
                    )

        st.markdown("#### 🧩 Additional Standard Clauses")
        c1, c2, c3, c4 = st.columns(4)
        opt_indemnity = c1.checkbox("Indemnification", value=True)
        opt_severability = c2.checkbox("Severability", value=True)
        opt_arbitration = c3.checkbox("Binding Arbitration", value=False)
        opt_force_majeure = c4.checkbox("Force Majeure", value=False)

        submitted = st.form_submit_button("⚡ Generate Agreement", use_container_width=True)

    if submitted:
        rendered = render_template(selected_template_name, form_values)
        
        # Inject additional selected clauses if needed
        extra_clauses_text = ""
        if opt_arbitration and "arbitration" not in rendered.lower():
            extra_clauses_text += f"\n\nADDITIONAL CLAUSE: BINDING ARBITRATION\nAny controversy or claim arising out of or relating to this Agreement shall be settled by binding arbitration in {form_values.get('jurisdiction', 'the prevailing jurisdiction')} under the commercial rules of the American Arbitration Association."
        if opt_force_majeure and "force majeure" not in rendered.lower():
            extra_clauses_text += "\n\nADDITIONAL CLAUSE: FORCE MAJEURE\nNeither party shall be held liable for failure or delay in performance resulting from acts beyond reasonable control, including acts of God, governmental orders, natural disasters, or telecommunication failures."
        
        st.session_state.current_document = rendered + extra_clauses_text
        st.session_state.current_doc_title = selected_template_name
        st.success(f"Generated {selected_template_name} successfully!")

# ==========================================
# MODE 2: AI CUSTOM CONTRACT DRAFTER
# ==========================================
elif navigation_mode == "🤖 AI Custom Drafter":
    st.subheader("🤖 AI Bespoke Legal Contract Drafter")
    st.write("Describe your specific agreement requirements, parties, and deal structure in natural language. Gemini AI will construct a tailored legal instrument.")

    col_ai1, col_ai2 = st.columns([2, 1])
    with col_ai1:
        custom_doc_type = st.text_input("Contract Title / Type", value="Software Licensing & Maintenance Agreement")
        custom_prompt = st.text_area(
            "Contract Requirements & Deal Terms",
            height=180,
            value="Licensor grants Licensee a non-exclusive, non-transferable enterprise license for cloud ERP software. Licensee pays $15,000 annually. SLA guarantees 99.9% uptime. Licensor provides 24-hour bug fixes for critical severity issues. Excludes right to sublicense or reverse-engineer.",
            help="Specify obligations, deliverables, pricing, warranties, liabilities, and special conditions."
        )
    with col_ai2:
        custom_jurisdiction = st.text_input("Governing Jurisdiction", value="Delaware, United States")
        st.markdown("**Include Recommended Clauses:**")
        add_confidentiality = st.checkbox("Confidentiality / NDA", value=True)
        add_ip = st.checkbox("Intellectual Property Assignment", value=True)
        add_indemnity = st.checkbox("Limitation of Liability & Indemnity", value=True)
        add_dispute = st.checkbox("Dispute Resolution / Court Venue", value=True)

    selected_clauses = []
    if add_confidentiality: selected_clauses.append("Confidentiality")
    if add_ip: selected_clauses.append("IP Rights and Ownership")
    if add_indemnity: selected_clauses.append("Limitation of Liability and Indemnification")
    if add_dispute: selected_clauses.append("Dispute Resolution and Governing Law")

    if st.button("🚀 Draft Contract with AI", use_container_width=True):
        with st.spinner("AI Legal Counsel is drafting the agreement..."):
            generated_doc, was_ai = generate_ai_contract(
                prompt=custom_prompt,
                document_type=custom_doc_type,
                jurisdiction=custom_jurisdiction,
                clauses=selected_clauses,
                api_key=active_api_key
            )
            st.session_state.current_document = generated_doc
            st.session_state.current_doc_title = custom_doc_type
            if was_ai:
                st.success("✨ Contract drafted via Google Gemini AI!")
            else:
                st.info("📄 Contract generated via Smart Synthesizer (Add Gemini API key in sidebar for generative deep drafting).")

# ==========================================
# MODE 3: CONTRACT RISK AUDITOR
# ==========================================
elif navigation_mode == "🔍 Contract Risk Auditor":
    st.subheader("🔍 AI Contract Risk & Vulnerability Auditor")
    st.write("Scan your draft or paste an existing legal document to identify vulnerabilities, missing protections, and ambiguous wording.")

    audit_input = st.text_area(
        "Paste Contract Text for Risk Audit",
        value=st.session_state.current_document if st.session_state.current_document else "",
        height=220,
        placeholder="Paste your full legal contract text here..."
    )

    if st.button("🛡️ Audit Contract for Legal Risks", use_container_width=True):
        if not audit_input.strip():
            st.warning("Please provide or generate a contract text to audit.")
        else:
            with st.spinner("Auditing contract clauses and risk posture..."):
                audit_report = audit_contract_risks(audit_input, api_key=active_api_key)
                st.markdown(audit_report)

# ==========================================
# MODE 4: PLAIN ENGLISH EXPLAINER
# ==========================================
elif navigation_mode == "📖 Plain English Explainer":
    st.subheader("📖 Legalese to Plain English Translator")
    st.write("Translate dense legal provisions, jargon, and obligations into plain, easily understandable terms for clients and stakeholders.")

    explain_input = st.text_area(
        "Paste Legal Text to Simplify",
        value=st.session_state.current_document if st.session_state.current_document else "",
        height=220,
        placeholder="Paste complex contract clauses here..."
    )

    if st.button("💡 Explain in Plain English", use_container_width=True):
        if not explain_input.strip():
            st.warning("Please enter legal text to explain.")
        else:
            with st.spinner("Translating legalese into plain English..."):
                plain_summary = explain_in_plain_english(explain_input, api_key=active_api_key)
                st.markdown(plain_summary)

# ==============================================================
# LIVE WORKSPACE & DOCUMENT EXPORT (ALWAYS ACCESSIBLE BELOW)
# ==============================================================
st.markdown("---")
st.subheader("📄 Document Workspace & Preview")

if st.session_state.current_document:
    # Allow user to edit document directly
    edited_document = st.text_area(
        "Live Document Editor (Editable)",
        value=st.session_state.current_document,
        height=350,
        key="doc_editor_area"
    )
    # Sync edits
    st.session_state.current_document = edited_document

    # Clause Refinement AI Assistant
    with st.expander("✨ Refine or Add Clauses with AI"):
        col_ref1, col_ref2 = st.columns([3, 1])
        with col_ref1:
            refine_inst = st.text_input("Refinement Prompt", placeholder="e.g. Add a 12-month non-compete clause, or increase payment late fee to 10%")
        with col_ref2:
            st.write("")
            st.write("")
            refine_btn = st.button("Apply AI Refinement")

        if refine_btn and refine_inst.strip():
            with st.spinner("Applying requested refinement..."):
                updated_doc, _ = refine_document_with_ai(
                    original_text=st.session_state.current_document,
                    instruction=refine_inst,
                    api_key=active_api_key
                )
                st.session_state.current_document = updated_doc
                st.rerun()

    # Export Buttons
    st.markdown("#### 📥 Export & Download")
    col_dl1, col_dl2, col_dl3 = st.columns(3)

    # 1. PDF Export
    with col_dl1:
        pdf_bytes = generate_legal_pdf(
            st.session_state.current_document,
            title=st.session_state.current_doc_title
        )
        st.download_button(
            label="📄 Download Formatted PDF",
            data=pdf_bytes,
            file_name=f"{st.session_state.current_doc_title.lower().replace(' ', '_')}_{datetime.today().strftime('%Y%m%d')}.pdf",
            mime="application/pdf",
            use_container_width=True
        )

    # 2. Markdown Export
    with col_dl2:
        st.download_button(
            label="📝 Download Markdown (.md)",
            data=st.session_state.current_document,
            file_name=f"{st.session_state.current_doc_title.lower().replace(' ', '_')}.md",
            mime="text/markdown",
            use_container_width=True
        )

    # 3. Plain Text Export
    with col_dl3:
        st.download_button(
            label="📋 Download Plain Text (.txt)",
            data=st.session_state.current_document,
            file_name=f"{st.session_state.current_doc_title.lower().replace(' ', '_')}.txt",
            mime="text/plain",
            use_container_width=True
        )

else:
    st.info("💡 No document currently active. Select a template above or use the AI Custom Drafter to generate your first legal agreement.")
