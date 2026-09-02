# CLAUDE.md

## Co to jest

Ansible + Kubernetes home-lab budowany od zera, jako projekt nauki DevOps. Poprzednia wersja: `/home/dawid/Dokumenty/nauka/infrastructure/infra` — zostaje jako archiwum, nie jest kontynuowana. Docelowo: 1 control plane + 2 workery (Debian), klaster k8s postawiony `kubeadm`, z docelowym wdrożeniem realnej aplikacji (`home-budget`).

**Hosty:** control plane `k8s-ctrl` (`10.0.0.20`) + workery `k8s-wk1` (`10.0.0.21`), `k8s-wk2` (`10.0.0.22`) — wszystkie trzy w `inventory`/`group_vars`/`host_vars`. Klaster k8s postawiony (`kubeadm init` + `kubeadm join`), CNI (Flannel) zainstalowane — wszystkie node'y `Ready`, klaster funkcjonalny end-to-end (roadmapa pkt 1-8 zrobione). Stan na Dzień 12 (patrz `Journal/infra-daily/12-08-2026.md`) — źródło prawdy dla aktualnego postępu to zawsze najnowszy plik w `Journal/infra-daily/`, ta linia bywa nieaktualizowana na bieżąco.

## Zasady pracy (WAŻNE — przestrzegaj zawsze)

- **Jeden mały, domknięty krok dziennie** (30-60 min pracy Dawida).
- **Dawid pisze kod. Claude robi code review — nie implementuje za Dawida.** Wyjątek: Dawid wyraźnie prosi o podpowiedź/rozwiązanie, bo utknął.
- Code review ma być konkretny: błędy, brak idempotencji, złe practices, alternatywne podejścia — nie tylko "wygląda dobrze". **Bez pytań kontrolnych / quizowych** ("zastanów się dlaczego...", "pytanie kontrolne: ..."). Dawid dopiero się uczy Ansible i Kubernetesa — nie zakładaj, że ma już wiedzę, którą można sprawdzić pytaniem. Zamiast pytania, od razu wytłumacz.
- Zadania na dany dzień (plik w `Journal/infra-daily/`) mają zawierać **dużo teorii, nie tylko listę komend do odpalenia**. Każdy krok diagnostyczny musi tłumaczyć, co oznacza jego wynik i jaką decyzję ten wynik wymusza (np. nie "sprawdź `apt policy containerd`", tylko też: jak czytać numer wersji, co oznacza `Installed: (none)`, jaka reguła decyzyjna z tego wynika). Cel: Dawid uczy się jednocześnie Ansible **i** samego Kubernetesa/mechanizmów pod spodem (CRI, cgroups, sieć itd.) — teoria k8s dla warstwy, którą się danego dnia konfiguruje, to osobna sekcja, nie dopisek do kroków Ansible.
- **Format instrukcji: opisowo, nie pytająco.** Zamiast "zastanów się, jakiego modułu użyć" — napisz wprost, jakiego modułu/komendy użyć i gdzie sprawdzić dokumentację. Jeśli jest kilka sensownych podejść (np. `hosts:` vs `run_once`+`delegate_to`, dwie role vs jedna z `when:`), opisz każde z konkretnymi wadami i zaletami, i wskaż rekomendację — decyzję i tak podejmuje Dawid, ale ma dostać materiał do wyboru, nie zagadkę do rozwiązania. To samo dotyczy teorii k8s: tłumacz mechanizm wprost (co się dzieje i dlaczego), nie prowadź do wniosku pytaniami naprowadzającymi. Dawid pisze kod sam — to nie znaczy, że ma znać Ansible/k8s od zera bez wytłumaczenia.
- Po zaakceptowanej zmianie: notatka w `@/home/dawid/Dokumenty/obsidian-vaults/Main/Journal/infra-daily/DD-MM-YYYY.md` — co zrobione, jakie koncepcje, co poprawione po review i dlaczego (nie tylko "co", ale "dlaczego było źle").
- Repo na razie lokalne, bez zdalnego repo na GitHubie — push dopiero gdy Dawid o to poprosi.
- Nie dodawaj funkcji/ficzerów wybiegających poza zadanie danego dnia — trzymaj się jednego kroku na raz.

## Postęp — źródło prawdy

Pełna historia dzień-po-dniu jest w `@/home/dawid/Dokumenty/obsidian-vaults/Main/Journal/infra-daily/` (jeden plik na dzień, format `DD-MM-YYYY.md`). Przed zaplanowaniem kolejnego kroku, przejrzyj tam ostatnie pliki, żeby wiedzieć co już jest zrobione i co było poprawiane w review.

## Roadmapa (kolejne kroki, orientacyjnie — jeden punkt to zwykle kilka dni pracy)

1. Inventory + `ansible.cfg` + pierwszy playbook: łączność SSH, `apt update` na jednym hoście.
2. Bootstrap hosta: user administracyjny, sudoers, klucz SSH (jako zmienna, nie hardkod), hardening SSH.
3. Zmienne: `group_vars`/`host_vars`, wersja k8s, CIDR sieci podów.
4. Rola `k8s_prereqs`: swap off, moduły jądra, sysctl.
5. Rola `containerd`: instalacja, config, `SystemdCgroup=true`, handlery.
6. Rola `k8s_packages`: repo pkgs.k8s.io, kubelet/kubeadm/kubectl, apt-hold.
7. ✅ `kubeadm init` na control plane + `kubeadm join` na workerach — zrobione (Dzień 9-10, `hostvars`/kolejność bloków `hosts:` zamiast `delegate_to`/`run_once`, patrz `Journal/infra-daily/07-08-2026.md` i `11-08-2026.md`).
8. ✅ CNI (Flannel) — zrobione (Dzień 11, moduł `kubernetes.core.k8s`, patrz `Journal/infra-daily/12-08-2026.md`). Przy okazji poprawiona latentna wada w roli `k8s_prereqs` z Dnia 4 (`modprobe` bez `persistent:` nie przetrwał restartu hosta).
9. Kubeconfig lokalnie, zarządzanie klastrem przez `kubectl` z własnej maszyny. (przygotowane na Dzień 12, `Journal/infra-daily/13-08-2026.md`, jeszcze nie zrobione)
10. Wdrożenie `home-budget` na klastrze (Deployment + Service).
11. Ingress + TLS.
12. Ansible Vault dla sekretów (klucz SSH, przyszłe tokeny) zamiast hardkodów.
13. Helm chart dla `home-budget` — dopiero po tym, jak aplikacja chodzi na klastrze ze zwykłych manifestów YAML (pkt 10-11), żeby najpierw poznać surowe obiekty k8s, zanim zostaną zawinięte w szablon. Helmfile świadomie pominięty — sensowny dopiero przy wielu środowiskach/chartach naraz, czego na razie nie ma.
14. Orkiestracja: `site.yml` (`import_playbook` spinający wszystkie dotychczasowe playbooki w jedną sekwencję) + CI: `ansible-lint` / `--syntax-check` w GitLab CI (po wypchnięciu repo na GitLab; docelowo rozszerzone o `helm lint`/build obrazu `home-budget`, więc ten punkt i tak wchodzi po Helm).
15. **MetalLB** — bare-metal odpowiednik `Service type: LoadBalancer` (w chmurze IP przydzielałby cloud provider, tu nie ma kto tego zrobić bez MetalLB). Naturalnie łączy się z istniejącym ingress-nginx z Dnia 15. Okazja, żeby zrozumieć mechanikę różnicy `ClusterIP`/`NodePort`/`LoadBalancer`, nie tylko nazwy.
16. **Monitoring** (`kube-prometheus-stack`: Prometheus + Grafana + Alertmanager) — czytanie metryk cAdvisor/kubelet, pisanie `ServiceMonitor`/`PodMonitor`, rozumienie resource requests/limits jako czegoś widocznego na wykresach, nie tylko liczb w manifeście.
17. **Storage** (`local-path-provisioner` albo Longhorn) — PV/PVC/StorageClass dla bazy danych `home-budget`, temat jeszcze nietknięty w dotychczasowej roadmapie.
18. **External Secrets Operator (ESO)** — zastąpienie dzisiejszego wzorca (Secrety tworzone przez rolę Ansible z `group_vars/all/vault.yml`, chart Helma tylko referencjonuje nazwę) kontrolerem synchronizującym Secrety z zewnętrznego źródła, bez ich obecności w Gicie nawet w postaci zaszyfrowanej. Naturalnie wchodzi po punkcie 14 (podłączenie `kubernetes.core.helm` do roli) — decyzja i uzasadnienie zapisane w `Journal/infra-daily/9_wrzesień/04-09-2026.md` (Dzień 22): ESO wybrany świadomie zamiast pośredniego kroku "rola nadal tworzy Secrety osobno", bo to wzorzec faktycznie stosowany w zespołach, więc bardziej opłaca się nauczyć go od razu niż wdrażać tymczasowe rozwiązanie po drodze.

Punkty 15-17 dodane 2026-08-31, żeby pogłębić K8s poza "aplikacja działa end-to-end" — kierunek: LB → monitoring → storage. Braki wiedzy z samego Linuksa (sieć, cgroups, `/proc`) wplatane punktowo przy odpowiednim temacie (np. iptables/ARP przy MetalLB, cgroups v2 przy monitoringu), a nie jako osobny wątek — tak jak już przy `containerd`/`k8s_prereqs`.

Roadmapa jest orientacyjna — priorytet ma rzeczywisty postęp z `Journal/infra-daily/`, nie sztywne trzymanie się kolejności, jeśli Dawid chce pogłębić jakiś temat dłużej.

## Zadanie na dziś

Zadania z każdego dnia będą na branchach, które odpowiadają zadaniu z pliku w `@/home/dawid/Dokumenty/obsidian-vaults/Main/Journal/infra-daily/`

Patrz najnowszy plik w `@/home/dawid/Dokumenty/obsidian-vaults/Main/Journal/infra-daily/9_wrzesień/04-09-2026` - feature/04-09-2026
